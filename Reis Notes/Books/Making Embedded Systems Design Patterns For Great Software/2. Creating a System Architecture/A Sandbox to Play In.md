- Want to keep algorithms as segregated as possible and the model-view-controller is a good option for this
- MVC:
	- Model is the algorithm
	- View is the interfaces to the user so they could be sensors or other I/O
	- Controller is the thing that works in between these two
- Example of this:
	- ![[Pasted image 20260910130654.png]]

- Sandbox enviorment is a good way to test the algorithm or model independently from the embedded stuff to ensure that the logic is correct
- Below is an example of a MVC with a sandbox:
	- ![[Pasted image 20260910130902.png]]

- Sometimes sandboxes can be time consuming to develop so be careful in the approach and whether something needs one or not