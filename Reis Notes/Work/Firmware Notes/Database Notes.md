### Directory:
 - src:
		- manually created
	- generated:
		- python script auto gen
	- protobuf:
		- layer underneath Change Of Value messages
		- Scripts for protobuf code generation as well as scripts to start a server (allows user to get/set target data via COV)
	- protobuf_interface:
		- Source code compiled in MCU image, for MCU to send COV messages

### Functions:
- **db_add_nvm_callbacks()**: relies on these callbacks to read from our non volatile memory. Takes in the driver nvm write and nvm read
- **db_init()**: initializes the database
- **db_restore_from_nvm()**: invokes nvm access drivers to fetch NVM values for the database
- **db_set()**: takes in the key and the value
- **db_get_value()**: takes in the key

### Flow Chart :

	PC -> protobuf(COV) -> Target MCU

- database.csv is the generated data map on the PC. This get sent as a typedef struct definitions and are on the target MCU

### Database Configuration
- To add an entry to the database you need to modify the database.csv
- `flags`: are used to specify which product line
- `ll`: is lower limit
- `ul`: is upper limit
- `name`: is the base for the final enum which can be found in db_defines_<flags_specific>.h
- `value`: default value
- `type`: type of value
- `units`: are engineering units (e.g. pascal)
- `tags`:
- `access`:

### General Notes:

- database_generate.py - takes in csv's to generate the .h/.c files (database.csv needs to not have spaces for parsing to work)

### Data Flow Example:

![[Pasted image 20260811155137.png]]