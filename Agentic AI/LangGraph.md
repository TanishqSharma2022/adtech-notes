
Langchain and LangGraph both are used to make agentic systems. The difference is just that LangGraph gives more control and is more low level than Langchain.

Chains vs Graphs.
In Chains it has a single linear control flow. 
![[Screenshot 2026-02-24 at 10.47.28 AM.png]]

But what we want is LLM to choose its own control flow.
Router is fixed steps and LLM chooses its own control flow like an if else
Fully Autonomous me LLM chooses everything whatever it wants to do. 
But this hinders reliability as fully autonomous does not work very well. This is where LangGraph comes in to give better control and reliability. 
![[Screenshot 2026-02-24 at 10.51.38 AM.png]]

### State
It is just the schema that serves as the input schema for all Nodes and Edges in the graph. We append to this state as we progress through the whole flow to keep track of where we are and what happened previously.
```python
class State(TypedDict):
	graph_state: str
```

### Node
It is just the work you do. 
``` python
def node_1(state):
	print("---Node 1---")
	return {"graph_state": state['graph_state'] +" I am"}
```

### Edges 
Defines the control flow. Where to go from a node. Can also be conditional edge. It is just function that defines where to go based on the nodes or which node to return basically. 

### Graph Construction

**StateGraph** class is what is used to build a graph. Initialize the graph with the state. Then add nodes. Then add edges. Then compile the graph. We also import START, END which are predefined as use them as starting and ending nodes for the graph.

```python
builder = StateGraph(State)
builder.add_node("node_1", node_1)
  
# Logic
builder.add_edge(START, "node_1")
builder.add_conditional_edges("node_1", decide_mood)
builder.add_edge("node_2", END)

# Add
graph = builder.compile()

# View
display(Image(graph.get_graph().draw_mermaid_png()))
```

### Graph Invocation
To invoke the graph, we can directly use the `invoke` method. 
```python
graph.invoke({"graph_state" : "Hi, this is Lance."})
```




### Chains

For you can directly get AIMessage and UserMessage. To make a chatbot, just keep appending all of this in a message list sequentially. 
```python
llm = ChatOpenAI(model='')
result = llm.invoke(message)
result, result.response_metadata
```

#### Binding Tools 

```python
@tool 
def multiply(a, b):
	 return a*b
	 
llm.bind_tools(multiply) # to bind to llm chains
graph.add_node(ToolNode(multiple)) # to bind to graph with ToolNode(import it)
```



#### Messages as States
We do not want to overwrite the list but append to it to keep the whole history of conversation. This leads us to reducer functions.
```python
class MessageState(TypedDict):
	messages: list[AnyMessage]
```

#### Reducer Functions

`add_messages` is the reducer function used to append the current message with the whole message list
```python 
add_messages(initial_list, new_message)
```
in LangGraph, there is already a `MessageStates` State, that we can import which has the `messages` list and an `add_messages` reducer function attached to directly use it.

### Tools in Graph
```python 
@tool 
def multiply(a, b):
	 return a*b
	 
graph.add_node(ToolNode(multiple)) # to bind to graph with ToolNode(import it)
graph.add_conditional_edge(tools_condition)
```