Date: 9/3/2026
Attendees: Roberto, Tats, Sean, Doug, Masa, Ken, Song, Makoto, Nirmal

Notes:
- Database we have a setup where there is only one function that will write the data
- Need to create a test case that will ensure that in one cycle which will cause two write operations on the same variable
- Database items maybe we can create a data flow diagram which we can then see who accesses the data and where the variables are shared
- Fear that a variable can get overwritten from another module. Is this even a problem? Is this something that we would have to prevent?
- Run a calculation to see how long the control thread takes
- Make sure that we have a method to return to normal operation even if we have a deadlock