- We might have to build and maintain a configuration tool similar to DDC to generate a configuration file
- DDC ties its application and the executable for the config generator to a zip file that it releases as it's application
- The config file is all that hydra will need
- Just a note if we merge the motor program files into this it would break DDC configuration tool because it pulls the motor values from those files
- Recommend that anything information that the product team likes to set for them to make that in their own tool or approval system then the configuration tool would just pull from that. So if there are issues we wont have to keep changing the tool and instead the data that it pulls from
- DDC pulls the motor data from somewhere 
- Hydra will only need 
- Can leverage serial plate website so that product
- When serial plate data gets made it gets pushed into mapics and is not a part number



**Design Proposition**:

Production Flow:
1. Scan Barcode at line to read in the code string
2. Run the config file generator on production computer

Product Team Responsibility:

Controls Team Responsibility:

Production Team Responsibility: