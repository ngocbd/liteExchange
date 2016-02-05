Licence : All samples are "Public Domain" code 
http://en.wikipedia.org/wiki/Public_domain_software

===========================================================================
			
**The project description and technical information :** A multithreaded limit order matching engine written in C++11 using FIX protocol 4.2 . 

- It targets both Linux ( tested on CentOS and Ubuntu ) and Windows systems ( tested on Windows 8.1).

- It uses STL,Boost and also platform specific APIs ( POSIX, a few NP POSIX calls and WindowsAPIs) in some places, QuickFix for FIX protocol.

- It supports GCC4.8 and MSVC120 (VS2013). See "source/compiler_portability" 

- Additionally used GoogleTest for unit testing, Bash for Linux test scripts and Powershell for Windows test scripts.

Detailed articles about the project and its implementation details will be available in my blog at www.nativecoding.wordpress.com.
						
**Limit orders and order matching engines :** For limit orders please see : https://en.wikipedia.org/wiki/Order_%28exchange%29#Limit_order

Basically a limit order is an order which you specify a maximum price for the security you want to buy. 
As for the terminology a limit order to buy is called a "bid" and a limit order to sell is called an "ask".

An order matching engine matches the ask orders and the bid orders. This implementation places the highest bid order on
top of bids table and places the lowest ask order on top of asks table. Then it chekcs the bids and asks and tries to 
match the orders. And then it sends status reports back to the owners of the orders. A status of an order can be :

- "accepted" : the server accepted the order and it will be processed
- "filled" : the order matched
- "partially filled" , meaning that some trading happened , but still more to googlecode
- "rejected" , if order type is not supported.
- "canceled" , if a client wants to cancel an order ,and if the order is canceld , the server informs the client

For general information about the trading systems and the order types , please see :
http://www.investopedia.com/university/intro-to-order-types/

**FIX ( Financial information exchange ) protocol :** It is a session based TCP protocol that carries financial security transcation data.

For more information , please see https://en.wikipedia.org/wiki/Financial_Information_eXchange .

For the time being, this projectis using opensource QuickFix engine and FIX specification 4.2.
	
**How multithreading is implemented for order matching:** The engine mainly consists of 2 parts : a FIX server/engine and the order matching layer.
The core of order matching layer is called a central order book, which keeps order books per security symbol.
Each order book has a table for bids and another table for asks. Briefly the multithreaded system works as below :

1. The FIX engine will listen for session requests from the clients, and if a session is established
then it listens for incoming ask/bid orders. If the order type is not supported. It sends "rejected" message.
	
2. Central Order book has a thread pool

		- The thread pool will have N SPSC queues for N threads ( N = num of symbol ).
		
		- Central Order book also has 1 MPMC queue for outgoing messages.
		
		- When a new message arrives ( new order, or cancel ) from the FIX engine, it will be submitted to corresponding thread`s queue in the thread pool of the central order book.
		
3. Each thread in the thread pool will get message from their SPSC queue in the thread pool , and add them to corresponding order queue which is used by only itself
and eventually trigger the order matching process for that queue. At the end of the order matching , worker threads will submit messages ( FILLED or PARTIALLY FILLED ) to the outgoing messages queue 

4. Outgoing message processor which is a MPMC queue will process the outgoing messages.

**Build dependencies :** For Linux , the project is built and tested with GCC4.8. As for Windows it is using MSVC1200(VS2013). In the libraries side :

- Boost 1.59 : using a compacted version by using the command below :

				bcp --boost=c:\boost_1_59_0 shared_ptr scoped_ptr any optional tokenizer format c:\boost
						
- QuickFix & its requirements : libxml2-devel & http://www.quickfixengine.org/quickfix/doc/html/install.html

**Runtime dependencies :** For Windows, you have to install MSVC120 runtime : https://www.microsoft.com/en-gb/download/details.aspx?id=40784
For Linux, you need GNU Lib C runtime and libxml2.
			
**How to build the project on Linux :**
	
	cd build/linux
	make debug  OR make release

**How to build the project on Linux using Netbeans 8.0.2 C++ IDE:**

	Open Netbeans.
	Open the project from the project directory. ( Choose "nbproject" directory )
	Build the project inside Netbeans IDE.

**How to build the project on Windows  :**
	
	You can build with Visual Studio 2013
	Go to "build/windows" directory
	Use SLN file to launch VS with the project

**Server parameters and running the matching engine :** The engine executable looks for "ome.ini" file. Here is the list of things you can set :

		- FILE_LOGGING_ENABLED							enables/disables logging
		- CONSOLE_OUTPUT_ENABLED						enables/disables output to stdout
		- CENTRAL_ORDER_BOOK_PIN_THREADS_TO_CORES		whether to pin threads of the threadpool to different CPU cores
		- HYPER_THREADING								if hyperthreading is off and pinning is on ,then it will pin threads to only cores with an even index
		- CENTRAL_ORDER_BOOK_QUEUE_SIZE_PER_THREAD		Queue size per worker thread in the central order book`s thread pool
		- LOG_BUFFER_SIZE								Maximum buffer size for the logging system as it is built on a ring buffer.
		
You will also need to have "quickfix_FIX42.xml" and "quickfix_server.cfg" files to be in the same directory with OME executable. You can find them in "bin" directory.


**Functional testing :** There is a prebuilt executable for both Linux and Windows which can send specified ask/bid orders to the order matching engine.
   
   Under "test_functional" directory :
   
		1. Modify test_data.txt which has the orders to send to the engine as you wish.
		
		2. Modify  the arrays declared on top of client_automated_test.sh/client_automated_test.ps1 script files in order to configure the number of clients. You should provide a name for each client.
		
		3. For firing Linux test client(s), you can use client_automated_test.sh Bash script.
		
		4. For firing Windows test client(s), you can use client_automated_test.bat which drives client_automated_test.ps1 Powershell script.
		
		5. After firing the script, it will be executing all orders in test_data.txt file per client that is declared in the script file.
		
**Unit testing :** The project uses GoogleTest 1.7. You can find a makefile and vcproj under "test_unit" directory.
	
**Building and running unit test on Linux :** You have to build and install Google Test 1.7 first , the instructions for CentOS and Ubuntu :
			
			$ wget http://googletest.googlecode.com/files/gtest-1.7.0.zip
			$ unzip gtest-1.7.0.zip
			$ cd gtest-1.7.0
			$ ./configure
			$ make
			$ sudo cp -a include/gtest /usr/include
			
			On CentOS
			$ sudo cp -a lib/.libs/* /usr/lib64
			
			On Ubuntu
			$ sudo cp -a lib/.libs/* /usr/lib/
			
			Then you can either use Makefile or Netbeans project files under "test_unit" directory.

**Building and running unit test on Windows :** You can use VisualStudio solution in "test_unit" directory.

**Source code and file/directory naming conventions :**
	
	directory names					lower_case_word
	file names 						lower_case_word
	include guards 					_HEADER_
	macros							UPPER_CASE_WORD
	enums							UPPER_CASE_WORD
	namespace names 				lower_case_word
	class names 					CamelCase
	method names 					pascalCase
	variable names					pascalCase
	member variables starts with	m_

**Source code indentations and new line usage :**
	
	Space based no tabs ( This needs to be setup in VS )
	By default Netbeans editor uses spaces for tabs
	Needs to be set in VS2013 : https://msdn.microsoft.com/en-gb/library/ms165330(v=vs.90).aspx
	New lines : Unix CR only ( \n ) , VS can handle it even though Windows is \r\n

**Warning level used for GCC :** -Wall

**Warning level used for MSVC :** /W3
		
**Precompiled header file usage :** On Windows , the project is using /FI ( Force include parameter, therefore no need to include the pch header everywhere ) and specified the pch header to be precompiled_header.h. Note that this breaks edit-and-continue in Visual Studio.
For Linux , there is pch rule to enable it in the makefile ( build/linux/Makefile) , but currently that rule is not being used since it doesn`t appear as it is doing much improvement as on Windows.

**Considerations for future :**

Concurrency : Lockfree containers

Memory : 3rd party memory allocators support : jemalloc, intelTBB, tcMalloc, Lockless

Compiler support : Use of CMake, upgrading supported compiler versions and enabling use of C++14/C++17, experimental Clang and IntelC++Compiler

Order matchinng : Adding other order types ( market orders, stop loss order) , order update and market data request support, TIF support

Exchange connectivity : Support ITCH ( London Stock Exchange & NASDAQ ) and/or FIX engines , interested in using Libtrade : https://github.com/libtrading/libtrading

Concurrency : Experiment order book processing on GPU with CUDA

New feature : Event broadcasting and sample feed handlers

New feature : Visualisation with OpenGL/Vulkan

OS Support : Solaris