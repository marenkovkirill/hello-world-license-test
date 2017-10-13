# RCML - Documentation for developers

- [ 1. Creating Own Modules for RCML](#1-creating-own-modules-for-rcml)
	- [1.1 Robot Module](#11-robot-module)	
		- [1.1.1 Robot Module Library API](#111-robot-module-library-api)
		- [1.1.2 Robot module API](#112-robot-module-api)
			- [1.1.2.1 The *getModuleInfo* Method](#1121-the-getmoduleinfo-method)
			- [1.1.2.2 The *prepare* Method](#1122-the-prepare-method)
			- [1.1.2.3 The *getFunctions* Method](#1123-the-getfunctions-method)
			- [1.1.2.4 The *getAxis* Method](#1124-the-getaxis-method)
			- [1.1.2.5 The *writePC* Method](#1125-the-writepc-method)
			- [1.1.2.6 The *init* Method](#1126-the-init-method)
			- [1.1.2.7 The *final* Method](#1127-the-final-method)
			- [1.1.2.8 The *readPC* Method](#1128-the-readpc-method)
			- [1.1.2.9 The *startProgram* Method](#1129-the-startprogram-method)
			- [1.1.2.10 The *getAviableRobots* Method](#11210-the-getaviablerobots-method)
			- [1.1.2.11 The *robotRequire* Method](#11211-the-robotrequire-method)
			- [1.1.2.12 The *robotFree* Method](#11212-the-robotfree-method)
			- [1.1.2.13 The *endProgram* Method](#11213-the-endprogram-method)
			- [1.1.2.14 The *destroy* Method](#11214-the-destroy-method)
		- [1.1.3 Robot API](#113-robot-api)		
			- [1.1.3.1 The *prepare* Method](#1131-the-prepare-method)
			- [1.1.3.2 The *getUniqName* Method](#1132-the-getuniqname-method)
			- [1.1.3.3 The *executeFunction* Method](#1133-the-executefunction-method)
			- [1.1.3.4 The *axisControl* Method ](#1134-the-axiscontrol-method)
	- [1.2 Function Module](#12-function-module)	
		- [1.2.1 Function module library API](#121-function-module-library-api)
		- [1.2.2 Function module API](#122-function-module-api)
	- [1.3 Control Module](#13-control-module)	
		- [1.3.1 Control module library API](#31-control-module-library-api)
		- [1.3.2 Control module API](#132-control-module-api)
		- [1.3.3 The *execute* Method](#133-the-execute-method)
	- [1.4 Choosing module](#14-choosing-module)	
		- [1.4.1 Chosing module library API](#141-chosing-module-library-api)
		- [1.4.2 Chosing module API](#142-chosing-module-api)
		- [1.4.3 The *makeChoise* Method](#143-the-makechoise-method)
- [2. About interface identifiers for modules](#2-about-interface-identifiers-for-modules)
- [3. Work with RCML statistics](#3-work-with-rcml-statistics)
	- [3.1 General information and recommendations](#31-general-information-and-recommendations)
	- [3.2 Structure of statistics database](#32-structure-of-statistics-database)

# 1 Creating Own Modules for RCML

All description of classes in *C++* programming language cited in this section, along with the appropriate header files, as well as examples of modules can be found in the relevant official repository [module_headers](https://github.com/RobotControlTechnologies/module_headers)

All types of modules in *RCML* environment are created by dynamic link libraries (dll files for the OS of Windows family, or so files for the OS of Linux family). A variety of APIs to interact with the robot are implemented by using them. They are described below.

Unification of the interaction interface at the software level provides the developer with advantages in choosing the hardware interfaces, hardware itself, communication methods, technologies, etc. in solving tasks of integration of its solutions with RCML environment.

The instructions below will be given with respect to C++ programming language, but you can use any other programming language to create modules, it is important that dynamic link libraries provided certain interfaces, similar to those used to implement in C++.

Regardless of the type of module, module library must export only two functions:

- A function that returns API version supported by the module;
- A function that returns a pointer to the descendant of the abstract class of the corresponding module type.

However, the names of these functions, as well as the result returned depend on the module type. These functions do not have parameters, and their names must be strictly defined.

These functions are called only once at the launch of the compiler or interpreter. Accordingly, it is guaranteed that the object with a pointer returned by this function, is always in the singular as part of a single instance of *RCML* environment.

**Attention!** Developer should remember that it is possible to run several interpreters and/or compillers at the same time to solve different tasks, so each laucnhed instance wiil call functions and critical methods for handling equipment such as *init, final, destroy* from the module file in the process of their work. According to the nuances of multithreading, it should be noted that their call can be simultaneous in time.

**Attention!** Also remember that when the compiler is running, the methods of the modules are called from the same thread, because the compiler is single-threaded When the interpreter methods are used, they can be called from different sources at arbitrary times, rather than strictly sequentially.

**All modules that work with the equipment must be thread-safe.**

It is highly recomended to use a *singletone* design pattern in the process of robot module developement, consider that the calls of the robot module methods can happen from different threads and even simultaneously.

## 1.1 Robot Module

As noted earlier, to be able to send commands to a real physical robot through *RCML*, this robot must be represented in *RCML* environment by robot module.

### 1.1.1 Robot Module Library API

Robot module library must export the following functions:
```
unsigned short getRobotModuleApiVersion();
RobotModule* getRobotModuleObject();
```
*getRobotModuleApiVersion* function returns the version of *API* of robot module in a given module as a positive integer. The best way to determine this function is to use the following code:
```
unsigned short getRobotModuleApiVersion() { return MODULE_API_VERSION; }
```
*MODULE_API_VERSION* constant is defined in *robot_module.h* linked file, which is required when compiling robot module. Thus, we can omit specifying a version manually, and it will match the module interface used.

*getRobotModuleObject* function returns a pointer to the object of the abstract class RobotModule, essentially describing the required robot module interface.

Hereinafter we will refer *RobotModule* class as the abstract robot module. Correspondingly, the class pointed by the returned pointer, must be inherited from it (the abstract robot module) and must re-define all virtual methods of this class, maintaining mode of application to them. This subsidiary class will be referred to as robot module.

### 1.1.2 Robot module API

RobotModule class description in *C++*:
```
class RobotModule {
 protected:
  RobotModule() {}

 public:
  // init
  virtual const struct ModuleInfo& getModuleInfo() = 0;
  virtual void prepare(colorPrintfModule_t *colorPrintf_p,
                       colorPrintfModuleVA_t *colorPrintfVA_p) = 0;

  // compiler only
  virtual FunctionData **getFunctions(unsigned int *count_functions) = 0;
  virtual AxisData **getAxis(unsigned int *count_axis) = 0;
  virtual void *writePC(unsigned int *buffer_length) = 0;

  // intepreter - devices
  virtual int init(initCallback_t& initCallback) = 0;
  virtual void final() = 0;

  // intepreter - program & lib
  virtual int readPC(int pc_index, void *buffer, unsigned int buffer_length) = 0;

  // intepreter - program
  virtual int startProgram(int run_index, int pc_index) = 0;
  virtual AviableRobotsResult *getAviableRobots(int run_index) = 0;
  virtual Robot *robotRequire(int run_index, Robot *robot) = 0;
  virtual void robotFree(int run_index, Robot *robot) = 0;
  virtual int endProgram(int run_index) = 0;

  // destructor
  virtual void destroy() = 0;
  virtual ~RobotModule() {}
};
```

#### 1.1.2.1 The *getModuleInfo* Method

*getModuleInfo* method must return *ModuleInfo* structure describing this module. This method can be called multiple times during one work session.

The previously mentioned *ModuleInfo* structure, describing the module is as follows:
```
struct ModuleInfo {
  char *iid;
enum Modes { PROD, SPEC } mode;
  unsigned short version;
  char *digest;
};
```
The following items are given in this structure:

- *iid*– identifier of the interface of this module. This line should not be longer than 32 bytes Read more in Section ["Additional Information About Interface Identifiers"](http://docs.rcml.tech/en#13-additional-information-about...).
- *mode* – type of version of this module file. It can be *PROD* – production version, or *SPEC* – specification version. Production version involves fully-featured module file, declaring module functions and contains the code for their execution. Specification version assumes that the module file only declares module functions in *RCML* environment, but does not include a code for their execution. Accordingly, production version of the module can also be used to compile *RCML* programs with this module and for their execution, while specification version can only be used for compilation. The interpreter will not work with specification version of the module. This mechanism allows developers of robot modules to declare robot module interface without transferring its master code. This can be useful when working with contractors that write software for this module. They may have means of verifying the proper use of the interface, but will not be able to get the real module.
- *version* – the current module version as a positive integer. If the module is not planned to be distributed through the Repository service, this field does not play a great role. But otherwise, you need to consider the fact that the Repository requires that the next version of the loadable module was higher than previously loaded one.
- *digest* – a pointer to the digital signature of the module developer. It is used only by the Repository service.

#### 1.1.2.2 The *prepare* Method

*prepare* method is called once immediately after the module is loaded into memory. Two pointers to system functions of *RCML* environment allowing the logged output of various data are sent as parameters of the method. Output from these procedures is sent to standard output and can be formatted and written to the relevant log files of *RCML* environment.

Functions, to which pointers are sent, have the following description:
```
void colorPrintfFromModuleVA(void *module, ConsoleColor colors, const char *mask, va_list args);
void colorPrintfFromModule(void *module, ConsoleColor colors, const char *mask,...);
```

A pointer to the current instance of robot module, i.e. this must be sent as *module* parameter. *ConsoleColor* structure must be sent as colors parameter. This parameter sets the output color at the console. The third parameter *mask* defines formatting mask similar to printf function from the standard library of *C++*. Further, depending on output function, according to the defined formatting mask, goes common enumeration of parameters or enumeration of parameters in the form of *va_list* structure from *std_arg.h*.

#### 1.1.2.3 The *getFunctions* Method

*getFunctions* method is called only by the compiler. It must return a pointer to an array of pointers to *FunctionData* structures, describing every available robot function. And this method takes one parameter by link – *count_functions*, which must include the number of array elements with pointers to FunctionData, see details below. If robot module does not provide any functions, this method must return *NULL* pointer, and write 0 to *count_functions* parameter.

*FunctionData* structure is used to describe each function of the robot available for use in *RCML* language. Description of the structure in C++:
```
structFunctionData {
enumParamTypes { STRING, FLOAT };

system_value command_index;
  unsigned int count_params;
  ParamTypes *params;
  const char *name;
  FunctionData() : command_index(0), count_params(0), params(NULL), name(NULL) {}
  FunctionData(system_value command_index, system_value count_params,ParamTypes *params, const char *name) : command_index(command_index),count_params(count_params),params(params),name(name) {}
};
```
The following items are given in this structure:

- *command_index* – a unique command identifier within the module. This value will further be used and sent to robot representation by the interpreter to indicate the command to be executed by a particular robot. Command index must be an integer greater than zero. Values of zero and negative values are reserved and are system values;
- *count_params* – a number of function parameters equal to the number of elements in params array;
- *params* – a pointer to the array of parameter types. Each array element must be represented by ParamTypes counting element: *STRING* – a line or *FLOAT* – a number. See detailed description of the mechanism of parameter transfer below;
- *name* – a pointer to the name of a function specified by *RCML* programmer to call this function.


#### 1.1.2.4 The *getAxis* Method

*getAxis* method is also called by the compiler only. It must return a pointer to an array of pointers to *AxisData* structures, describing each available axis of robot control for hand control mode. This method, similar to the previous one, takes one parameter by link – *count_axis*, which must include the number of array elements with pointers to *AxisData*, see details below. If robot module does not support hand control mode, this method must return *NULL* pointer, and write 0 to count_axis parameter.

In turn, *AxisData* structure is used to describe each robot control axis available for use in hand_control system function in *RCML*. 

Description of this structure in *C++*:
```
struct AxisData {
  system_value axis_index;
  variable_value upper_value;
  variable_value lower_value;
  const char *name;
  AxisData() : axis_index(0), upper_value(0), lower_value(0), name(NULL) {}
  AxisData(system_value axis_index, variable_value upper_value,variable_value lower_value, const char *name) : axis_index(axis_index),upper_value(upper_value),lower_value(lower_value),name(name) {}
};
```
The following items are given in this structure:

- *axis_index* – a unique axis identifier within the module. This value will further be used and sent to robot representation by the interpreter to indicate the axis value to be changed. Similar to unique command identifiers, axis index must be an integer greater than zero;
- *upper_value* – the upper limit value, which can be transferred for a given axis;
- *lower_value* – the lower limit value, which can be transferred for a given axis;
- *name* – a pointer to the axis name specified by *RCML* programmer to call hand_control system function to link to this axis.


#### 1.1.2.5 The *writePC* Method

*writePC* method is called only by the compiler. This method allows the module to write (to the file of the program compiled) arbitrary data with the pointer to their range to be returned by the method as a result, and the length of data recorded in *buffer_length* parameter transferred by link.

#### 1.1.2.6 The *init* Method

*init* method is used for robot module to execute procedures necessary to get properly started, for example, reading the configuration file, establishing communication with robots, etc. The method is called once by the interpreter only at launch, immediately after loading dll library in memory and receiving a pointer to an instance of robot module. This method must return 0, if there were no errors when loading, and calling environment can continue operation. Otherwise – any other value different from 0, and calling program accordingly ends operation generating an error on impossibility to initialize this module.

Setting *initCallback* function:

```
const RobotInfoResult* initCallback(unsigned size, const RobotInfo robots[]);
```

Parameter *size* is the size of the array of structures *RobotInfo*, link on which is sent in the second parameter *robots*.

Description of the *RobotInfo* structure:

```
struct RobotInfo {
  const char* uniqueName;
  const char* serialNumber;
};
```

Following structure describes every robot connected to the module and includes two parameters:
- *uniqueName* - pointer to a line - a unique name of robot, which is used to display system messages related to this robot (to simplify the identification of the robot);
- *serialNumber* - pointer to a line - a serial number of the robot


The function *initCallback* compares the serial numbers of the robots with those specified in the current RCML license and returns a pointer to the *RobotInfoResult* structure.

Structure of *RobotInfoResult*:

```
struct RobotInfoResult {
  const bool* serialNumbers;
  const char* signature;
};
```

*True* values in the boolean array *serialNumbers* indicate the serial numbers that passed the verification. The sequence number of the element with the value *True* is equal to the number in the structure *RobotInfo* containing the serial number that passed the verification. The *signature* property contains a pointer to the *RCML* digital signature for this module.

#### 1.1.2.7 The *final* Method 

*final* method is used for robot module to execute procedures necessary for correct completion of work, for example, saving some intermediate values, switching off of the robots, etc. This method, like init method, is called only once by the interpreter upon completion of the main program execution.

#### 1.1.2.8 The *readPC* Method

The *readPC* method is only called by the interpreter when loading another program or library into memory; if it contains data recorded by this module, a pointer to the domain of these data will be passed to the buffer parameter, and the size of this data will be passed to the *buffer_length* parameter. The *pc_index* parameter is a unique index of the downloaded *PCode* file (this index is used in other methods, see later). If there is no data recorded by the module, this method will not be called.

#### 1.1.2.9 The *startProgram* Method 

The *startProgram* method is only called by the interpreter before executing the next *RCML* program, the *run_index* parameter will be a unique number – program run ID, and the *pc_index* parameter will be a unique index of the loaded *PCode* file, from which the program is executed. The file can be loaded once, but the program can be invoked several times; it can even be invoked in parallel threads. If this method returns 0, the program will be passed for invocation. Otherwise, getting any other value different from 0, the *RCML* environment will not execute the program, and will show an error stating that the module prohibits running the current program. This method allows restricting program invocation, based on the data obtained in the *readPC* method.

#### 1.1.2.10 The *getAviableRobots* Method 

This method is called when the robot with choise module is activated. This method is used to provide information about free robots to the choise modules, so that they can select the robot accordingly.

The *getAviableRobots* method returns a pointer to the *AviableRobotsResult* structure, which contains information about all the robots that are available in this module at the time of request.

The description of previously mentioned structure *AviableRobotsResult* that describes the list of available modules:

```
struct AviableRobotsResult {
  Robot **robots;
  unsigned int count_robots;
  AviableRobotsResult(Robot **robots, unsigned int count_robots) : robots(robots), count_robots(count_robots) {}
  void destroy() { delete this; }
  ~AviableRobotsResult() { delete[] robots; }
};
```

Description of elements used in this structure:

- *robots* – array of pointers to free robots;
- *count_robots* – number of elements in *robots* array.

#### 1.1.2.11 The *robotRequire* Method

The *robotRequire* method is only called by the interpreter at the point of program execution, when the necessity occurs to use a robot of some class that is associated with the robot module. The robot parameter contains a pointer previously obtained from the *getAviableRobots* method, to an instance of the class inherited from abstract class Robot mapped to the particular requested physical robot. The robot module should check that the requested robot is available, and if it is, activate the requested robot, and return a pointer to it, otherwise return *NULL*, which means that this robot cannot be activated. If the robot parameter is equal to *NULL*, the robot module may return any available robot or, if there are no available robots. 

In the first case, the *robot* parameter specifies the pointer obtained from the *getAvailableRobots* method, on a particular robot. The robot module should check that the requested robot is free, and if this is the case, use the requested robot and return the pointer to it, otherwise return * NULL *, which means that this robot can not be used.

In the second case, the *robot* parameter will be *NULL*, and the robot module can return any free robot or *NULL* if there are no free robots.

**Important**! It should be noted that this method should not delay execution to wait for an available robot; this will be done by internal mechanisms of the *RCML* environment.

#### 1.1.2.12 The *robotFree* Method

*robotFree* method is called only by the interpreter when a certain engaged robot has ceased to be necessary for a specific program and can be released. A pointer that was previously obtained from robotRequire method will be sent as a parameter. It is guaranteed that this pointer is not changed during program execution.

#### 1.1.2.13 The *endProgram* Method

The *endProgram* method is also called only by the interpreter after execution of the *RCML* program, the same run ID of the program will be passed as the *run_index* parameter, as the one passed for calling the *startProgram* method, which describes the program. If this method returns 0, it is considered that the program has been executed without errors, and its completion code will not be changed. If any other number is returned, the program is considered to have worked with errors, and the completion code will be different from 0. This method allows the module to report at the end of the program that the execution of this program with this module was not correct, and there were errors during execution.

#### 1.1.2.14 The *destroy* Method

*destroy* method is the alias of robot module destructor. It must call a destructor of robot module object, if necessary, and release engaged resources, since *RCML* environment does not call the destructor method due to the nature of the code generated by different compilers. This method is called once by the compiler and interpreter upon completion of work before unloading of the module from memory by a certain instance of the interpreter or compiler (by calling *FreeLibrary* functions).

### 1.1.3 Robot API

Hereinafter robot representation means a class to which instance a pointer is returned by *robotRequire* method. And this class must be inherited from the abstract *Robot* class, hereinafter referred to as an abstract robot representation. Essentially, an instance of robot representation is a representation of a physical robot in *RCML* environment, which was mentioned in description of the language syntax and which *RCML* environment interacts with a particular real robot through. Robot representation must implement all public methods of abstract robot representation. 

All methods of *Robot* class are called only by *RCML* interpreter.

Description of the abstract Robot class in *C++*:
```
class Robot {
 protected:
  Robot() {}

 public:
  virtual void prepare(colorPrintfRobot_t *colorPrintf_p,
                       colorPrintfRobotVA_t *colorPrintfVA_p) = 0;
  virtual const char *getUniqName() = 0;
  virtual FunctionResult *executeFunction(int run_index, CommandMode mode,
                                          system_value command_index,
                                          void **args) = 0;
  virtual void axisControl(system_value axis_index, variable_value value) = 0;
  virtual ~Robot() {}
};
```

#### 1.1.3.1 The *prepare* Method

*prepare* method is called only by the interpreter immediately after receiving a pointer to an instance of robot representation (through robotRequire module method), by analogy with the same method in the module, this method provides access to logging means in *RCML* environment. Pointers to functions with the following descriptions are sent to this method as parameters:

```
void colorPrintfFromRobotVA(void *robot, const char *uniq_name, ConsoleColor colors, const char *mask, va_list args);
void colorPrintfFromRobot(void *robot, const char *uniq_name, ConsoleColor colors, const char *mask, ...);
```

A pointer to the current instance of robot representation, i.e., this must be sent as robot parameter. *uniq_name* parameter can sent a pointer to a line – robot name, or it can be *NULL* value, then robot name will be replaced by its serial number at the output. *ConsoleColor* structure must be sent as colors parameter. The third parameter (*mask*) defines formatting mask similar to printf function from the standard library of *C++*. Further, depending on output function, according to the defined formatting mask, goes common enumeration of parameters or enumeration of parameters in the form of va_list structure from std_arg.h.

#### 1.1.3.2 The *getUniqName* Method

The method *getUniqName* returns a pointer to a string - the unique name of the robot used to log actions. Called many times for a session.

#### 1.1.3.3 The *executeFunction* Method

*executeFunction* method is called by the interpreter when a certain physical robot matched to this robot representation, must execute the desired function. In this case, one of the values of CommandMode enumeration will be sent to mode parameter. Through this parameter, robot representation can learn how the current function is exactly called. However, the actions of robot representation have no impact on waiting or not waiting of execution of the called function by *RCML* environment. The value of this parameter can be as follows:

- *wait* – the command is executed waiting for the execution;
- *not_wait* – the command is executed not waiting for the execution;
- *package* – the command is sent in the package, and is not the last command in the package;
- *end_of_package_wait* – the last command in the package; the package is executed waiting for the execution;
- *end_of_package_no_wait* – the last command in the package; the package is executed not waiting for the execution.

A unique function identifier will be sent to *command_index* parameter. A pointer to an array of parameters of called functions will be sent to args parameter. The number of elements in this array will be equal to the number of parameters takes by this function, i.e., the number of parameters specified in count_params property of the instance of *FunctionData* structure corresponding to this function, which, as mentioned earlier, *RCML* environment will receive by calling *getFunctions* method. The sequence of parameter values in the array is direct, just as they were specified in a text file of a program in *RCML*, and just as the types of these parameters in params property were specified. A pointer to the parameter value which must receive text data, will point to a line (char *), and, respectively, a pointer to the parameter value receiving numerical data – to the number of variable_value type (double *). This method must return the function result, which is a pointer to FunctionResult structure. If the called function takes no parameters, *NULL* will be sent to args parameter of this method.

Description of *FunctionResult* in C++:
```
class FunctionResult {
 public:
  enum Types { EXCEPTION, VALUE };
 private:
  Types type;
  variable_value result;

public:
  FunctionResult(Types type) : type(type), result(0.0f) {}
  FunctionResult(Types type, variable_value result) : type(type), result(result) {}
  virtual Types getType() { return type; }
  virtual variable_value getResult() { return result; }
  virtual void destroy() { delete this; }
  virtual ~FunctionResult(){};
};
```
The following items are given in this structure:

- *type* – the type of value returned by robot function. If this type is *EXCEPTION*, it is considered that the function threw an exception that can be handled by try operator, otherwise it is considered that the function returned the value;
- *result* – the value returned by robot function; it will be read only if type is not *EXCEPTION*. This value may be just a number.

This method can be called specifying one of the following system values (named constants) as the value of command_index parameter, while NULL value will be sent to args parameter:

- *ROBOT_COMMAND_FREE* – is sent when this particular robot will be no more needed to the program. It should be noted that this parameter applies only to notify a certain robot that it will be further released, but it is not a command to release the robot for robot module. To release the robot, robotFree method of robot module is called;
- *ROBOT_COMMAND_HAND_CONTROL_BEGIN* – preparing a robot to switch to hand control mode; after processing this command, the commands will no more be sent to the robot, and axis values will be sent through axisControl method;
- *ROBOT_COMMAND_HAND_CONTROL_END* – exiting hand control mode.


#### 1.1.3.4 The *axisControl* Method 

*axisControl* method is called by the interpreter only in hand control mode. A unique identifier of robot axis from AxisData structure is sent as the first parameter of this method. A new value for this axis is sent as the second parameter. A mechanism of transfer is described in Section ["Value Transfer Principle"](http://docs.rcml.tech/en#4-2-value-transfer-principle).

### 1.2 Function Module

Function modules are used to extend the set of system functions of RCML environment (do not confuse with functions of system module). Because module functions can be created in traditional programming languages and tools, it is possible to integrate RCML environment with other software and technologies, and hence integrate it with robotics through RCML environment.

By its structure and principles of interaction, function modules are very similar to robot modules, so it is recommended to read the previous section.

#### 1.2.1 Function module library API

Dynamic link library of function module must export the following functions:
```
unsigned short getFunctionModuleApiVersion();
FunctionModule *getFunctionModuleObject();
```
Hereinafter we will refer FunctionModule class as the abstract function module. Correspondingly, the class pointed by the returned pointer, must be inherited from it and must re-define all virtual methods of this FunctionModule class, maintaining mode of application to them. This subsidiary class will be referred to as function module.

#### 1.2.2 Function module API

Description of *FunctionModule* class in *C++*:
```
class FunctionModule {
 protected:
  FunctionModule() {}

 public:
  // init
  virtual const struct ModuleInfo& getModuleInfo() = 0;
  virtual void prepare(colorPrintfModule_t *colorPrintf_p, colorPrintfModuleVA_t *colorPrintfVA_p) = 0;

  // compiler only
  virtual FunctionData **getFunctions(unsigned int *count_functions) = 0;
  virtual void *writePC(unsigned int *buffer_length) = 0;

  // intepreter - devices
  virtual int init() = 0;
  virtual void final() = 0;
  
  // intepreter - program & lib
  virtual int readPC(int pc_index, void *buffer, unsigned int buffer_length) = 0;

  // intepreter - program
  virtual int startProgram(int run_index, int pc_index) = 0;
  virtual FunctionResult *executeFunction(system_value function_index, void **args) = 0;
  virtual int endProgram(int run_index) = 0;

  // destructor
  virtual void destroy() = 0;
  virtual ~FunctionModule() {}
};
```
The *getModuleInfo, prepare, writePC, readPC, init, final, startProgram, endProgram* and *destroy* methods are completely similar to the homonymous methods of the robot module.

*getFunctions* method is similar to the same method of robot module; when it is called, function module must return a list of functions that it provides for calling from RCML environment.

*executeFunction* method is similar to the same method of robot representation, but the method of this type of modules has no *mode* parameter and when it (method) is called, the robot does not execute the function, but control is simply passed to the appropriate function of function module.

### 1.3 Control Module

Control modules are used to connect control devices to *RCML* environment and can only be used in the system function of switching to hand control mode. By its structure, control modules are very similar to robot modules, so it is recommended to first read Section ["Robot Module"](#1-1-robot-module).

#### 1.3.1 Control module library API

Control module library must export the following function:
```
unsigned short getControlModuleApiVersion();
ControlModule *getControlModuleObject();
```
Hereinafter we will refer *ControlModule* class describing the interface of control module as the abstract control module. Correspondingly, the class pointed by the returned pointer, must be inherited from it and must re-define all virtual methods of this *ControlModule* class, maintaining mode of application to them. This subsidiary class will be referred to as control module.

#### 1.3.2 Control module API

Description of *ControlModule class* in *C++*:
```
class ControlModule {
 protected:
  ControlModule() {}

 public:
  // init
  virtual const struct ModuleInfo& getModuleInfo() = 0;
  virtual void prepare(colorPrintfModule_t *colorPrintf_p, colorPrintfModuleVA_t *colorPrintfVA_p) = 0;

  // compiler only
  virtual AxisData **getAxis(unsigned int *count_axis) = 0;
  virtual void *writePC(unsigned int *buffer_length) = 0;

  // intepreter - devices
  virtual int init() = 0;
  virtual void execute(sendAxisState_t sendAxisState) = 0;
  virtual void final() = 0;

  // intepreter - program & lib
  virtual int readPC(int pc_index, void *buffer, unsigned int buffer_length) = 0;

  // intepreter - program
  virtual int startProgram(int run_index, int pc_index) = 0;
  virtual int endProgram(int run_index) = 0;

  // destructor
  virtual void destroy() = 0;
  virtual ~ControlModule() {}
};
```
The *getModuleInfo, prepare, writePC, readPC, init, final, startProgram, endProgram* and *destroy* methods are completely similar to the homonymous methods of the robot module.

*getAxis* method is similar to the same method in robot module, but when calling, control module must return a list of control axes that it provides for linking in the call of *hand_control* system function.

#### 1.3.3 The *execute* Method

*execute* method is called by the interpreter when switching to hand control mode. A pointer to the function of sendAxisState_t type will be sent as the only parameter of this method, defined as follows:
```
typedef void (*sendAxisState_t)(ControlModule *, system_value, variable_value);
```
This function must be called each time when the control device changes a value of the control axis. A pointer to this control module is sent as the first parameter of this function, followed by a unique axis identifier that was specified in the data returned by *getAxis* method. A new value of this axis is sent as the last parameter.

### 1.4 Choosing module

The choosing modules provide algorithms for decision-making. Like other modules, the choosing modules have methods that are common to all modules, as first described in the Section ["Robot Module"](#1-1-robot-module).

#### 1.4.1 Chosing module library API

The library of the choosing module should export the following functions:
```
unsigned short getChoiceModuleApiVersion();
ControlModule *getChoiceModuleObject();
```
Further, the abstract control module will be understood as the *ChoiceModule* class, which describes the interface of a choosing module, and, accordingly, the class to which the returned pointer is pointing should inherit from it, and override all virtual methods of this class, preserving the mode of calling then; this child class will be further referred to as the statistics module.

#### 1.4.2 Chosing module API

*ChoiceModule* class description in C++:
```
 class ChoiceModule {
 protected:
  ChoiceModule() {}

 public:
  // init
  virtual const struct ModuleInfo& getModuleInfo() = 0;
  virtual void prepare(colorPrintfModule_t *colorPrintf_p, colorPrintfModuleVA_t *colorPrintfVA_p) = 0;

  // compiler only
  virtual void *writePC(unsigned int *buffer_length) = 0;

  // intepreter - devices
  virtual int init() = 0;
  virtual void final() = 0;

  // intepreter - program & lib
  virtual int readPC(int pc_index, void *buffer, unsigned int buffer_length) = 0;

  // intepreter - program
  virtual int startProgram(int run_index, int pc_index) = 0;
  virtual const ChoiceRobotData *makeChoice(const ChoiceFunctionData** function_data, unsigned int count_functions, const ChoiceRobotData** robots_data, unsigned int count_robots) = 0;
  virtual int endProgram(int run_index) = 0;

  // destructor
  virtual void destroy() = 0;
  virtual ~ChoiceModule() {}
};
```
The *getModuleInfo, prepare, writePC, readPC, init, final, startProgram, endProgram* and *destroy* methods are completely similar to the homonymous methods of the robot module.

#### 1.4.3 The *makeChoise* Method

The *makeChoice* method is called when the choosing module is required to make a decision about choosing a robot. It is in this method that the robot choosing logic has been implemented. The following parameters are passed to the input of this method:

- *function_data* - the array of pointers to the *ChoiceFunctionData* structures that represents the sequence of functions of the robot to be chosen.
- *count_functions* - the number of elements in the *function_data array*;
- *robots_data* - the array of pointers to the *ChoiceRobotData* structures; this array has a list of pointers to descriptors of available robots obtained from robot modules;
- *count_robots* - the number of elements in the *robots_data* array.

The *makeChoice* method should return a pointer to the description of the *ChoiceRobotData* presentation of the chosen robot. If the decision about choosing the robot cannot be made, the *makeChoice* method should return *NULL*.

**Important**! If the choosing module is supposed to work with the *RCML* statistics database, section ["Working with RCML statistics"](http://docs.rcml.tech/en#14-working-with-rcml-statistics) has an important addition about it.
```
struct ChoiceFunctionData {
  enum CallType {
    Unknow = 0,
    Guarante = 1,
    Probably = 2
  };
  const char *name;
  const char *context_hash;
  system_value position;
  CallType call_type;
}; 
```
In this structure, the following properties have been marked:
- *name* - the name of the function that will be called for the chosen robot;
- *context_hash* - the hash of the *RCML* file of the program represented as a string of binary data, where a call to this function has been detected;
- *position* - a unique position of calling this function in the file of the *RCML* program. This property should not be confused with the number of the line where this function is called in the program source code;
- *call_type* - the type of function call that may take the following values:
  - *Unknow* - the *RCML* interpreter can't compute call of this function, it is an exceptional situation equivalent to internal error;
  - *Guarante* - the function is guaranteed to be called, i.e. it does not depend on any conditions;
  - *Probably* - the function will probably be called, i.e. in the path to calling this function, there are branching conditions that may result in a failure to call this function. It should be noted that in this context, the condition is not the *if* operator, but the conditional transfer - an equivalent to the *jz* assembler command with a computed condition. Unconditional transfers do not fall into this definition.

The *ChoiceRobotData* structure that describes the representation of the robot has the following С++description:
```
struct ChoiceRobotData {
  const ChoiceModuleData *module_data;
  const char *robot_uid;
};
```
In this structure, the following properties have been marked:

- *module_data* - a pointer to the *ChoiceModuleData* structure that describes the robot module;
- *robot_uid* - the name of the robot that must be unique within the robot module, however, it may not necessarily be so. This field may also be an empty string "".

The structure of *ChoiceModuleData* is:
```
struct ChoiceModuleData {
  const char *iid;
  const char *hash;
  unsigned short version;
};
```
It contains the following elements:

- *iid* - the interface module *ID* as a string;
- *hash* - the hash of the file module in the form of a string representation of binary data;
- *version* - the version of the module.

# 2 About interface identifiers for modules

The string, the pointer to which is stored in the *iid* property of the *ModuleInfo* structure returned by the *getModuleInfo* method, regardless of the module, is a unique identifier of the module's interface version, as well as IID in RCML libraries.

However, the unique identifier without damage can still coincide for modules of different types.

# 3 Work with RCML statistics

## 3.1 General information and recommendations

*RCML* stores statistics in external database file, which is specified in the configuration file *config.ini*.

The database engine * SQLite * ([http://www.sqlite.org/] (http://www.sqlite.org/)) is used to work with the database, so there is no delimitation of rights and access to the database. Therefore, to exclude database access conflicts for all non-RCML software, including robot selection modules, it is recommended that you access the database only in read mode without modifying the data.

It is important to note that the *RCML* statistics are written in the *runtime* mode, i.e. constantly, so it is not recommended to cache requests to the database, so as not to lose the relevance of the data. Also it is not recommended to keep the database statistics file open at all times.

### 3.2 Structure of statistics database

![](images/1.png)

Graphical representation of the structure of the statistical database available on the picture above.

*function_calls* table includes following fields:

- *id* - unique identifier of the record, primary key;
- *robot_id* - the identifier of the robot that performed this function, the foreign key, refers to the *id* field in the table *robot_uids*;
- *function_id* - the identifier of the executed function, the foreign key, refers to the *id* field in the table *functions*;
- *run_id* - identifier of launched *RCML* program, foreign key, refers to the *id* field in the table *runs*;
- *start* - the timestamp of the start of the function execution, the number of microseconds since the start of the *RCML* run of the program in which the function described by this record was called;
- *end* - the timestamp of the end of the function execution, the number of microseconds since the start of the *RCML* run of the program.

The program or library in which the robot function is called within the terminology of the statistical database is called the context of the execution of the function, i.e. the place where the function is called, which determines the conditions for its execution.

A table storing information about contexts is called *contexts* and contains the following fields:
- *id* - unique identifier of the context, primary key;
- *filename* - name of the context file;
- *hash* - a hash from a context file in the form of a string representation of binary data;
- *iid* - interface context identifier, in the form of a string, is specified only for contexts that are *RCML* libraries;
- *version* - the version of the context, is specified only for contexts that are *RCML* libraries.


The *functions* table reflects the position of a particular call of the robot function in the context, has the following fields:

- *id* - unique identifier of the location of the function call, primary key;
- *context_id* - the context identifier in which the function is called, the foreign key, refers to the *id* field in the table *contexts*;
- *name* - the name of the called function;
- *position* - unique position of the function call within the context. This property should not be confused with the line number in which this function is called in the source code of the *RCML* program.

The *runs* table reflects the startup timeline for executing those contexts of functions that are *RCML* programs. This table contains:

- * id * - the unique identifier for starting the context for execution, the primary key;
- *context_id* - the context identifier in which the function is called, the foreign key, refers to the *id* field in the table *contexts*;
- *run_at* - the timestamp of the start *RCML* of the program, reflecting the number of microseconds that have passed since the beginning of the 1900.01.01 00: 00: 00.000.000 period in the local time zone of the machine.

In addition to the contexts of functions, there are sources of functions. The source of the functions is *RCML* program or module (in this case the robot module) that stores the code of the called function. With this *RCML* program can be both a source and a context of functions, because it can contain robot functions that cause other robot functions.

A table describing the sources of functions is called *sources* and contains the following fields:

- *id* - unique source identifier, primary key;
- *type* - type of the source, takes the following values:
	- *1* - *RCML* library
	- *2* - robot module;
- *hash* - hash from the source file as a string representation of binary data;
- *iid* - interface identifier of the source, in the form of a line, is filled only for libraries and robot modules;
- *version* - version of the source of the functions.

The list of involved robots for execution of called functions is stored in the table *robot_uids*, which has the following fields:

- *id* - unique robot identifier, primary key;
- *source_id* - source identifier, in this case the source can only be a robot module, a foreign key, refers to the *id* field in the *sources* table;
- *uid* - the name of the robot, which should be recommended to make unique, but it may not be such.
	
