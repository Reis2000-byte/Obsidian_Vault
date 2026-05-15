Date: 4/16/2026

Goal: Learn about microchip dev tools. Focus seems to be on the vscode extension

- Microchip decided to move towards an open source approach. They will be working towards developing extensions for popular open source IDEs
- Could use Data Visualizer coming up to tune motors
- Goal is to make it look like VScode NOT mplabx in vscode

- Support tickets are for discussion. Error logs are for one way bug filing
- The LLM used inside of microchips key that they provide only uses the logic from the data sheets and their own DOS. It will not steal your data
- No virtual file system

- MPlabX will be shelved. All new support and features will be for the vscode extensions
- LSP allows it so that when there is a compiler change they don't have to do anything or just change the LSP. Before they had to build it into MPLabX
- There is a setting for the CMake that it will allow you to edit the CMake files BUT the tool will generate it for us

- Debugging:
	- Vscode tool allows use for GDB and mbdcore
	- Also works to use with a jlink

- You can cross develop between vscode AND MPLabX
	- Dont Change:
		- SRC
		- COmpiler
- Every time that a compiler setting is changed then a build will run. It will re-build the ninja and cmake files. Goal for microchip is that we don't even know that we are using cmake

- Build Flow:
	- Cmake -> Ninja -> elf
- mplab_backend is the part that runs in the backend that is the OG "mplabX". This closes every time vscode closes or restarts

- Postbuild Steps:
	- Located in the project properties overview section at the bottom
	- Can add steps that occur after the build occurs
- Mplab Code Completion:
	- When using the microchip project it will prompt us to disable intellisense
	- They but heads an it is better to use the micrchip provided one when developing with their tools
- .mplab folder
	- If you ever want to get rid of everything from the vscode extensions and everything that it installs then all you have to do is delete the folder
- Long Path Error:
	- Had to enable in windows
	- Also will have to enable it in every single compiler
	- It is a command in the command palatte
- Debugging:
	- IO View command: IO view. Lets you look at your IO outputs
	- Can also save all of the values to an excel file
	- Eclipse Memory Inspector