This document aims to help and explain how to use gsoap compiler and librairies over ONVIF API to manage ONVIF complient IP camera with C/C++ programming langage.
It supposes that you will read links added in it and that you already have an idea of the discussion between ONVIF client and server. Otherwise read ONVIF.txt .
Also we recommand you to read LIBS.txt if you want to understand what are the global content and use of principals libs in the gsoap package.
Any information included in this document has to be considered as understood by it writter. 

This document doen't focus on the development of the main application and problems linked to it.

**************************************************
WRITTEN BY : Nadir GHENNAM for OTODO SAS
**************************************************

0 - Index 

ONVIF = Open Network Video Interface Forum
SOAP = Simple Object Access Protocol
XML = eXtensible Markup Language
WSDL = Web Services Description Language
IPC = IP Camera

1 - Global understanding

This part is dedicated on explaining globally how gsoap works, what is ONVIF and their specifications. Many links in the part "Links and notes" are aimed to make you easier the reseach on understanding these tools.

**** WHAT IS ONVIF ****

ONVIF provides and promotes standardized interfaces for effective interoperability of IP-based physical security products. 
ONVIF utilizes IT industry technologies including SOAP, RTP, and Motion JPEG, MPEG-4, H.264 video codecs and H.265 video codecs
An ONVIF profile has a fixed and comprehensive set of features that enable a functional product to be developed solely on the profile specification.
An ONVIF profile has mandatory as well as conditional features, which are features that should be implemented by an ONVIF conformant device or client if it supports that feature in any way, including any proprietary way. (cf Profil S doc)

More specificly ONVIF uses SOAP command to do interactions between client and server. They provide APIs for each categories of commands. Basicly theses APIs are extracted from differents specifications.
Commands are SOAP, it is a messaging standard. SOAP uses an XML data format to declare its request and response messages, relying on XML Schema and other technologies to enforce the structure of its payloads.
In a deeper point of view, as we are using web services we can add that the format of XML files are WSDL (WSDL is specific type of XML document which describes the web service).


Now we brievly understand what is ONVIF, we want to build an app that can manage one or more IP Camera (IPC), considered as server, and a client. We choose C++ thus we need gSOAP toolkit.

**** WHAT IS GSOAP ****
gSOAP is a C and C++ software development toolkit for SOAP/XML web services and generic XML data bindings.
Given a set of C/C++ type declarations, the compiler-based gSOAP tools generate serialization routines in source code for efficient XML serialization of the specified C and C++ data structures.
The main goal of gSOAP is to make easier for programer to focus on the structure of their system rather than focusing on XML and SOAP request.

Furthermore, the gSOAP tools can be just as easily used to develop C/C++ applications that efficiently consume and produce XML by leveraging XML data bindings for C/C++ based on XML schemas.
Basically, an XML schema has an equivalent set of C/C++ data types for the components described by the schema.
So XML schema strings are just C/C++ strings, XML schema enumerations are C/C++ enums, XML schema complex types are just structs and classes in C/C++, and so on.
This enhances the reliability and safety of XML applications, because type-safe serializable C/C++ data types are serialized and validated in XML automatically.

This XML data binding means that your XML data is simply represented as C/C++ data. Reading and writing XML is a lot easier than using a DOM or SAX library for XML. 
This is not more expensive or more complex than it sounds. In fact, the generated XML serializers are very efficient to parse and validate XML and may run more than 30 times faster than validating XML parsers such as Apache Xerces C++.

In summary, gSOAP offers a type-safe and transparent approach to develop XML applications that has proven to be quicker to develop (by auto-coding), 
safer (by XML validation and type-safety), more reliable (by auto-generation of XML test messages and warnings), and higher performing (by efficient serializers and XML parsers generated in C/C++), compared to DOM and SAX libraries.


At this point we understand what gSOAP is : we will be able to generate libs and source code made of XML schema files and directly use it in our applications. It is made to make us only focus on the programmation of our application.
But we need to go deeper to understand more how gSOAP works, so we download it (https://sourceforge.net/projects/gsoap2/files/latest/download). We highly recommand to explore the downloaded file to see waht gSOAP is made of ...
In ~gsoap-2.8/gsoap/bin/win64/ : we have 2 .exe which are very important.
 
 * wsdl2h consumes WSDL and XSD schema files to converts them to C/C++ source code to implement XML messaging infrastructures. This frees the developer to focus on application functionality rather than on infrastructure.
   More specifically, the wsdl2h tool consumes WSDLs to generate a C or C++ interface header file, which uses a developer-friendly C/C++ header file syntax. 
   This allows developers to inspect Web services and XML schemas from a functionality point of view, rather than getting bogged down into the underlying SOAP-based infrastructure details of WSDLs and XSDs.
 
 * soapcpp2 generates all the Web service binding source code with XML serializers necessary to quickly develop server-side and client-side Web service APIs.
   The soapcpp2 tool can also be used to produce, rather than consume, WSDL and XSD files to deploy XML Web services or to develop XML applications. This approach allows the deployment of legacy C/C++ applications as services. 
   Simply describe the Web API in a C or C++ interface header file for the soapcpp2 tool to generate the C/C++ source code that glues everything together.

The source code of wsdl2h is located in gsoap-2.8/gsoap/wsld. The source code of soapcpp2 is located in gsoap-2.8/gsoap/src
In ~gsoap-2.8/gsoap/plugins ; Plugins to enhance the capabilities of the engine and to support WS protocols such as WS-Security, WS-Addressing, WS-ReliableMessaging, and WS-Discovery. Code in C and C++ .

[This section may evolve by the time of a better understanding]


With all of that being said we can now start to use gSOAP oriented for ONVIF.

2 - First steps - Before starting to code our ONVIF application

In this part we will focus on creating the files we need to code our application thanks to gsoap compilater. We assume that you are using a Linux based OS. 
Download the gsoap file (https://sourceforge.net/projects/gsoap2/files/latest/download). 

**** Installing gSOAP ****
To install gSOAP it is highly recommanded to have Flex, Bison and OpenSSL. 
Here are commands to do so according to gSOAP developper : 

sudo apt-get install flex bison
sudo apt-get install libssl-dev

Install the gSOAP software on Unix/Linux systems as follows:
./configure
make
sudo make install

For the last step we use admin (root) permissions using sudo. To install the executables in a local folder, say in $HOME/bin without requiring root access, use:
./configure                                         
make                                                
sudo make install exec_prefix=$HOME

Once done we are ready to use gSOAP tools. 

**** Converting ONVIF SOAP commands to C/C++ ****

wsld2h consumes WSDL/SOAP commands from ONVIF APIs to generates C/C++ interface files as headers. To do so, we use the typemap.dat file to map XSD schema types to C++ types and to shortern XML namespaces and datatypes
(basicly by using prefixes). This file is located at :  ~gsoap-2.8/gsoap/

Make sure to place typemap.dat in the current directory where you want to run wsdl2h and then execute the following command : 
wsdl2h -O4 -P -x -o onvif.h \http://www.onvif.org/onvif/ver10/device/wsdl/devicemgmt.wsdl \http://www.onvif.org/onvif/ver10/events/wsdl/event.wsdl \http://www.onvif.org/onvif/ver10/deviceio.wsdl \http://www.onvif.org/onvif/ver20/imaging/wsdl/imaging.wsdl \
http://www.onvif.org/onvif/ver10/media/wsdl/media.wsdl \ http://www.onvif.org/onvif/ver20/ptz/wsdl/ptz.wsdl \ http://www.onvif.org/onvif/ver10/network/wsdl/remotediscovery.wsdl \http://www.onvif.org/ver10/advancedsecurity/wsdl/advancedsecurity.wsdl

-O4 : optimize the output by "schema cycling" to remove unused schema
-P : remove base class xsd__anyType in the generated code (unused by ONVIF)
-x : removes the unnecessary generated code for the extensibility elements xsd:any and attributes xsd:anyAttribue, since we do not need to support these in general, except for some specific cases see further below.
-o : saves the binding interface code to the header file onvif.h
All links in the command are ONVIF API according to categories.
We let the reader the freedom to read the user guide to discover more about options of wsdl2h.
 
The onvif.h file contains all the details pertaining the ONVIF services in a developer-friendly format. It specifies the ONVIF data types used and the ONVIF Web services operations available.
This step requires wsdl2h with https enabled,run "make -f MakefileManual secure" in the gsoap/wsdl directory to build wsdl2h with https enabled. (you'll need first to compile soapcpp2)

In onvif.h : change #import "wsdd10.h" to #import "wsdd5.h" as Onvif uses WS-Adressing 2005/08 (in wsdd10.h it assumes WS-Adressing 2004/08)
To support WS-Discovery and WS-Security make sure that onvif.h has :
#import "wsdd5.h"
#import "wsse.h"

******************************************* WARNING ****************************************************
Only have wsa.h or wsa5.h for WS-Adressing because wsdd5.h imports wsa5.h to prevent compilation errors
********************************************************************************************************

Now we can use soapcpp2 tool to generate C++ proxy class and the databinding source code from onvif.h for our application :
soapcpp2 -2 -C -I ~/gsoap-2.8/gsoap/import -I ~/gsoap-2.8/gsoap -j -x onvif.h

-2 : forces SOAP 1.2
-C : generates client code without service code (as the IPC is already ONVIF complient)
 -j : generates C++ proxy classes (easier to use than classic classes)
-x : omits the generation of sample XML messages (which are a lot!)
-I : sets the import path to the ~/gsoap-2.8/gsoap/import directory in the gSOAP source code tree
We let the reader the freedom to read the user guide to discover more about options of soapcpp2.

To support client-side WS-Discovery operations, we run soapcpp2 as follows as documented here:
soapcpp2 -a -x -L -pwsdd -I ~/gsoap-2.8/gsoap/import ~/gsoap-2.8/gsoap/import/wsdd5.h

This generates wsddClient.cpp, which we will use later to compile with the project.


That's it ! you are now ready to start your application and focus on the structure of it and just use libs code generated !


3 - Links and notes 

- https://www.onvif.org/profiles/specifications/specification-history/june-2022/ (also useful https://www.onvif.org/specs/DocMap.html)
List of all available and most recent specifications on ONVIF

- https://www.onvif.org/specs/2206/ONVIF-Core-Spec-v2206.pdf
The ONVIF Core Specification aims to standardize the network interface of network video products. It defines a network video communication framework based on relevant IETF and Web Services standards 
including security and IP configuration requirements. (IP configuration, Device discovery, Device management,Media configuration, Real time viewing, Event handling, PTZ camera control, Video analytics, Security)


- https://www.onvif.org/wp-content/uploads/2016/12/ONVIF_WG-APG-Application_Programmers_Guide-1.pdf
The ONVIF Application Programmer's Guide. This document provides information about the use of the ONVIF standard from a programmer’s perspective complement to Core Specification and Test Specification.
Overview in pseudo code of intercations with ONVIF.

- https://www.onvif.org/wp-content/uploads/2019/12/ONVIF_Profile_-S_Specification_v1-3.pdf
Profile S specification.

- https://www.genivia.com/doc/guide/html/index.html#start
Genivia's (gSOAP creators) User Guide. Big document that explain in detail options and how works gSOAP.

- https://www.genivia.com/examples/onvif/index.html#Smart_Serialization_of_xsd:duration
Genivia's tutorial for ONVIF client and server.

- https://www.genivia.com/tutorials.html
Genevia's generic tutorial of usage of their toolkit. 

- https://www.genivia.com/downloads.html
Genivia download page

- https://www.genivia.com/docs.html
Genevia specification main page

- https://www.genivia.com/dev.html#overview
Get started with gSOAP, still lot of explaination but with more examples.

- https://www.genivia.com/doc/wsdd/html/wsdd_0.html
Page about WS-Discovery
