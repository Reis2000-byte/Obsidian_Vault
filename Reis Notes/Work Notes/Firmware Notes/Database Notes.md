- Directory:
	- src:
		- manually created
		- 
	- generated:
		- python script auto gen
	- protobuf:
		- layer underneath Change Of Value messages
		- Scripts for protobuf code generation as well as scripts to start a server (allows user to get/set target data via COV)
	- protobuf_interface:
		- Source code compiled in MCU image, for MCU to send COV messages

Functions:
- **db_add_nvm_callbacks()**: relies on these callbacks to read from our non volatile memory. Takes in the driver nvm write and nvm read
- **db_init()**: initializes the database
- **db_restore_from_nvm()**: invokes nvm access drivers to fetch NVM values for the database
- **db_set()**: takes in the key and the value
- **db_get_value()**: takes in the key
- 