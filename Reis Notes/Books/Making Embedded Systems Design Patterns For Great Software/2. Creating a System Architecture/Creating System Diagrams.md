
- Similar to hardware and schematics system diagrams can help show how the software will work together
- Four types of diagrams for this:
	- context diagrams
	- block diagrams
	- organigrams
	- layering diagrams

## Context Diagrams
- The first drawing is an overview of how our diagram will fit into the world at large
- Below is an example of a context diagram:
	![[Pasted image 20260813092631.png]]

- The goal here is to show a high-level context of how the system will be used by the customer
- Focus on the relationship between our device, users, servers, other devices, and other entities
- Questions to ask when making Context diagrams:
	- What problem is it solving? 
	- What are the inputs and outputs to my system? 
	- What are the inputs and outputs to the users? 
	- What are the inputs and outputs to the system at large?
-  Focus on the use cases and the world-facing interactions
- This diagram helps with defining system requirements and envision likely changes
- Helps keeping the scope in mind when diving into the software

## The Block Diagram
- Straight forward at the start because we are considering the physical elements of the system which helps us tie it to object oriented programming
- Hardware block diagrams are normally given by the hardware engineer to help show the correlation with your block diagram
- Below is an example of the two diagrams and how the work with each other:
	![[Pasted image 20260813093326.png]]

- It is important to separate communication methods from external components because as we add chips we will see some more critical components of our design
- Try to keep this simple and keep the schematic details out to the best of our abilities
- Next step is to add some higher level functionality. What is each external chip used for?
- Things to add to this for example:
	- databases
	- buffering systems
	- command handlers
	- algorithms
	- state machines

- After adding some of the items the block diagram can look like the following:
	![[Pasted image 20260813093945.png]]
Tip:
- If we are trying to learn a codebase, get a list of the code files. Try to connect them and see how they work with each other to better understand the code.

## Organigram
- This one looks like and organizational chart. Try to think of it like an org chart at a company
- Start with a main and trickle down. Think using inits() in the main and how we have to instantiate devices and what those connect to.
- Below is an example of this:
	![[Pasted image 20260813094600.png]]

- Below is an example of what it looks like when we use a shared resource to make the software more complex but avoid hardware cost:
	![[Pasted image 20260813094954.png]]

- Each time we add something we then have to consider how all the pieces connected work together making the system less robust

- Note: If we want to understand a code base act like a debugger. Start with the `main()` and step into interesting functions trying to not get too lost. Might take a few reps but the goal is to get a fell for where the flow of the code is going and how it gets there

## Layering Diagram
- This drawing looks for layers and represents objects by their estimated size
- Start at the bottom of the page and draw boxes for the things that go off the processor (like communication boxes)
- Next add all components that use the lowest layer. If there are multiple users of this then make the box larger
- Keep in mind if more things use something it becomes more complex
- Eventually this diagram shows you where the layers in our code are
- The goal of the diagram is to look for such points and think about the implications of combining the boxes in various ways
- If we have a group of several modules touching the same thing we might want to break that one apart
- Below is an example of this diagram:
	![[Pasted image 20260813100330.png]]