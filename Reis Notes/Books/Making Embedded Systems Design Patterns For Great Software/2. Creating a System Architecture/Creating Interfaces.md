- Initialization functions will need to be called multiple times in comparison to drivers
- High level functions should call these initialization functions to keep the modules consolidated
- Should version each piece of code that gets built
- Hold back gold plating everything until you define everything that the interface has to work with
- Use extern and static for variable scoping similar to object oriented private and public variables
- Static acts differently when used inside a function and outside of it

- Design pattern Singleton:
	- used when it is important that a class have exactly one instance
	- The singleton intercepts the request for a new object and will not create on if one is already made
	- Example in C++:
```
		class Singleton {
		public:
		  static Singleton* GetInstance() {
		    if (instance_ == NULL) {
		      instance_ = new Singleton;
		    }
		    return instance_;
		  }
		protected:
		  Singleton(); // no one can create this except itself
		
		private:
		  static Singleton* instance_; // the one single instance
		};
		
		// Define the one and only Singleton pointer
		Singleton* Singleton::instance_ = nullptr;
```
- Singleton can be used in logging to ensure that we only have one log instance
- Also helps with lazy allocation and initialization so that modules that are never used never consume resources

- Sharing private globals:
	- sometimes you will need to create a back door to the information
	- Instead of removing the static keyword we can instead return a pointer to a private variable:
```
static struct sLogStruct gLogData;
struct sLogStruct* LogInternalState() {
  return &gLogData;
}
```
- This isn't always the best so best to guard it when in normal production:
```
static struct sLogStruct gLogData;
struct sLogStruct* LogInternalState() {
#if PRODUCTION
  #error "Internal state of logging protected!"
#else
  return &gLogData;
#endif /* PRODUCTION */
}
```
- As we keep creating interfaces we need to keep in mind ways to verify our system