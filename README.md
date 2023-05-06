Download Link: https://assignmentchef.com/product/solved-ecs150-project4
<br>
You will be continuing the development of your virtual machine. You will be adding the ability to mount and manipulate a FAT file system image. Except were specified in this description the API will remain as stated in the Project 2 &amp; 3 descriptions. Files that are opened in the virtual machine will now be opened on the FAT image instead of in the host file system. The reading and writing of the files will affect the FAT image with the exception of file descriptor 0 – 2, which will still affect the standard in, out, and error files. The fatgen103.pdf in the resources is the Microsoft white paper on FAT. You do not need to fully implement all possible FAT file systems. You may make the following assumptions:

<ol>

 <li>All images have 512 byte sectors</li>

 <li>All images are FAT 16</li>

 <li>Long file name entries will contiguously proceed the corresponding short file name entry</li>

</ol>

You must implement the ability to create, read and write files that are in the root directory and to open and read the root directory. You must be able to read all of the short file name entries (long file name entries can be skipped) in the root directory.




The communication between the virtual machine and machine has changed slightly. The machine will be more responsive now, and the time in ticks passed to MachineInitialize will only affect the time for the machine to detect the virtual machine has terminated.




A working example of the vm and apps can be found in /home/cjnitta/ecs150. The vm syntax is vm [options] appname [appargs]. The possible options for vm are -t, -h, -m, -s

and -f; -t specifies the tick time in millisecond, -h specifies the size of the heap this is the size of the VM_MEMORY_POOL_ID_SYSTEM memory pool, -m specifies the

machine timeout/responsiveness in milliseconds, -s specifies the size of the shared memory used between the virtual machine and machine, and -f specifies the FAT image to be mounted. By default the times are set to 10ms, for debugging purposes you can increase these values to slow the running of the vm. The size of the heap by default is 16MiB, and the shared memory is 16KiB. The default FAT image is fat.ima. When specifying the application name the ./ should be prepended otherwise vm may fail to load the shared object file. In addition you will find several programs that will help create and manipulate the fat images in the /home/cjnitta/ecs150 directory.




Several functions have been provided for you in the VirtualMachineUtils.c that you might find useful, not all will be necessary: VMDateTime,

VMFileSystemValidPathName, VMFileSystemIsRelativePath,

VMFileSystemIsAbsolutePath, VMFileSystemGetAbsolutePath,

VMFileSystemPathIsOnMount, VMFileSystemDirectoryFromFullPath, VMFileSystemFileFromFullPath, VMFileSystemConsolidatePath, VMFileSystemSimplifyPath, VMFileSystemRelativePath.




The function specifications for both the virtual machine and machine are provided in the subsequent pages.




You should avoid using existing source code as a primer that is currently available on the Internet. You <strong>must</strong> specify in your readme file any sources of code that you or your partner have viewed to help you complete this project. All class projects will be submitted to MOSS to determine if pairs of students have excessively collaborated with other pairs. Excessive collaboration, or failure to list external code sources will result in the matter being transferred to Student Judicial Affairs.




<strong>Extra Credit: </strong>

Add support for subdirectories, this includes creating, deleting, and modifying files and directories anywhere in the directory structure. Implement the VMDirectoryUnlink and VMDirectoryCreate functions. Add support for long file names, the long file names

should be both read correctly from directories as well as created when new files are added to the mounted file system.

VMStart – Start the virtual machine.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMStart(<strong>int</strong> tickms, TVMMemorySize heapsize,  <strong>int</strong> machinetickms, TVMMemorySize sharedsize, <strong>const</strong> <strong>char</strong> *mount,  <strong>int</strong> argc, <strong>char</strong> *argv[]);

<h1>Description</h1>

VMStart() starts the virtual machine by loading the module specified by <em>argv</em>[0]. The <em>argc</em> and <em>argv</em> are passed directly into the VMMain() function that exists in the loaded module. The time in milliseconds of the virtual machine tick is specified by the <em>tickms</em> parameter, the machine responsiveness is specified by the <em>machinetickms</em>. The heap size

of the virtual machine is specified by <em>heapsize</em>. The heap is accessed by the applications through the VM_MEMORY_POOL_ID_SYSTEM memory pool. The size of the shared memory space between the virtual machine and the machine is specified by the <em>sharedsize</em>. The name of the FAT file system image is specified by <em>mount</em>.

<h1>Return Value</h1>

Upon successful loading and running of the VMMain() function, VMStart() will return VM_STATUS_SUCCESS after VMMain() returns. If the module fails to load, or the module does not contain a VMMain() function, or if the <em>mount</em> FAT file system image cannot be loaded, VM_STATUS_FAILURE is returned.







VMFileOpen    Opens and possibly creates a file in the mounted FAT file system.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMFileOpen(<strong>const</strong> <strong>char</strong> *filename, <strong>int</strong> flags, <strong>int</strong> mode,  <strong>int</strong> *filedescriptor);

<h1>Description</h1>

VMFileOpen() attempts to open the file specified by <em>filename</em>, using the flags specified by <em>flags</em> parameter, and mode specified by <em>mode</em> parameter. The file descriptor of the newly opened file will be placed in the location specified by <em>filedescriptor</em>. The flags and mode values follow the same format as that of open system call. The filedescriptor returned can be used in subsequent calls to VMFileClose(), VMFileRead(),

VMFileWrite(), and VMFileSeek(). When a thread calls VMFileOpen() it blocks in the wait state VM_THREAD_STATE_WAITING until the either successful or unsuccessful opening of the file is completed. In order to prevent confusion with standard in, out and error, the file descriptor returned in the location specified by <em>filedescriptor</em> will be 3 or larger.

<h1>Return Value</h1>

Upon successful opening of the file, VMFileOpen() returns VM_STATUS_SUCCESS, upon failure VMFileOpen() returns VM_STATUS_FAILURE. If either <em>filename</em> or <em>filedescriptor</em> are NULL, VMFileOpen() returns

VM_STATUS_ERROR_INVALID_PARAMETER.




VMFileClose     Closes a file that was previously opened in the mounted FAT file system.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMFileClose(<strong>int</strong> filedescriptor);

<h1>Description</h1>

VMFileClose() closes a file previously opened with a call to VMFileOpen().When a thread calls VMFileClose() it blocks in the wait state

VM_THREAD_STATE_WAITING until the either successful or unsuccessful closing of the file is completed.

<h1>Return Value</h1>

Upon successful closing of the file VMFileClose() returns VM_STATUS_SUCCESS, upon failure VMFileClose() returns VM_STATUS_FAILURE.







VMFileRead   Reads data from a file.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMFileRead(<strong>int</strong> filedescriptor, <strong>void</strong> *data, <strong>int</strong> *length);

<h1>Description</h1>

VMFileRead() attempts to read the number of bytes specified in the integer referenced by <em>length</em> into the location specified by <em>data</em> from the file specified by <em>filedescriptor</em>. The <em>filedescriptor</em> should have been obtained by a previous call to VMFileOpen(). The actual number of bytes transferred by the read will be updated in the <em>length</em> location. When a

thread calls VMFileRead() it blocks in the wait state VM_THREAD_STATE_WAITING until the either successful or unsuccessful reading of the file is completed. If the filedescriptor is less than 3 the reading will be attempted on the main file system, if the value of filedescriptor is 3 or larger the reading will be attempted on the mounted FAT file system file.

<h1>Return Value</h1>

Upon successful reading from the file, VMFileRead() returns VM_STATUS_SUCCESS, upon failure VMFileRead() returns VM_STATUS_FAILURE. If <em>data</em> or <em>length</em>

parameters are NULL, VMFileRead() returns

VM_STATUS_ERROR_INVALID_PARAMETER.




VMFileWrite Writes data to a file.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMFileWrite(<strong>int</strong> filedescriptor, <strong>void</strong> *data, <strong>int</strong> *length);

<h1>Description</h1>

VMFileWrite() attempts to write the number of bytes specified in the integer referenced by <em>length</em> from the location specified by <em>data</em> to the file specified by <em>filedescriptor</em>. The <em>filedescriptor</em> should have been obtained by a previous call to VMFileOpen(). The actual number of bytes transferred by the write will be updated in the <em>length</em> location. When a thread calls VMFileWrite() it blocks in the wait state

VM_THREAD_STATE_WAITING until the either successful or unsuccessful writing of the file is completed. If the <em>filedescriptor</em> is less than 3 the writing will be attempted on the main file system, if the value of <em>filedescriptor</em> is 3 or larger the writing will be attempted on the mounted FAT file system file.

<h1>Return Value</h1>

Upon successful writing from the file, VMFileWrite() returns VM_STATUS_SUCCESS, upon failure VMFileWrite() returns VM_STATUS_FAILURE. If <em>data</em> or <em>length</em>

parameters are NULL, VMFileWrite() returns

VM_STATUS_ERROR_INVALID_PARAMETER.







VMFileSeek Seeks within a file.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMFileSeek(<strong>int</strong> filedescriptor, <strong>int</strong> offset, <strong>int</strong> whence,  <strong>int</strong> *newoffset);

<h1>Description</h1>

VMFileSeek() attempts to seek the number of bytes specified by <em>offset</em> from the location specified by <em>whence</em> in the file specified by <em>filedescriptor</em>. The <em>filedescriptor</em> should have been obtained by a previous call to VMFileOpen(). The new offset placed in the <em>newoffset</em> location if the parameter is not NULL. When a thread calls VMFileSeek() it blocks in the wait state VM_THREAD_STATE_WAITING until the either successful or unsuccessful seeking in the file is completed. If the <em>filedescriptor</em> is less than 3 the seeking will be attempted on the main file system, if the value of <em>filedescriptor</em> is 3 or larger the seeking will be attempted on the mounted FAT file system file.

<h1>Return Value</h1>

Upon successful seeking in the file, VMFileSeek () returns VM_STATUS_SUCCESS, upon failure VMFileSeek() returns VM_STATUS_FAILURE.




<strong><em> </em></strong>













<strong><em>Name </em></strong>

VMDirectoryOpen – Opens a directory for reading in the mounted FAT file system.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMDirectoryOpen(<strong>const</strong> <strong>char</strong> *dirname, <strong>int</strong> *dirdescriptor);

<h1>Description</h1>

VMDirectoryOpen() attempts to open the directory specified by <em>dirname</em>. The directory descriptor of the newly opened directory will be placed in the location specified by <em>dirdescriptor</em>. The directory descriptor returned can be used in subsequent calls to VMDirectoryClose(), VMDirectoryRead(), and VMDirectoryRewind(). When a thread calls VMDirectoryOpen() it must block in the wait state

VM_THREAD_STATE_WAITING if the opening of the directory cannot be completed immediately. In order to prevent confusion with standard in, out and error, the file descriptor returned in the location specified by <em>dirdescriptor</em> will be 3 or larger.

<h1>Return Value</h1>

Upon successful opening of the directory, VMDirectoryOpen() returns

VM_STATUS_SUCCESS, upon failure VMDirectoryOpen() returns

VM_STATUS_FAILURE. If either <em>dirname</em> or <em>dirdescriptor</em> are NULL,

VMDirectoryOpen() returns VM_STATUS_ERROR_INVALID_PARAMETER.







VMDirectoryClose – Closes a directory that was previously opened in the mounted FAT file system.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMDirectoryClose(<strong>int</strong> dirdescriptor);

<h1>Description</h1>

VMDirectoryClose() closes a directory previously opened with a call to

VMDirectoryOpen().When a thread calls VMDirectoryClose() it blocks in the wait state VM_THREAD_STATE_WAITING if the closing of the directory cannot be completed immediately.

<h1>Return Value</h1>

Upon successful closing of the directory VMDirectoryClose() returns

VM_STATUS_SUCCESS, upon failure VMDirectoryClose() returns VM_STATUS_FAILURE.




VMDirectoryRead – Reads a directory entry from a directory in the mounted FAT file system.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMDirectoryRead(<strong>int</strong> dirdescriptor,

SVMDirectoryEntryRef dirent);

<h1>Description</h1>

VMDirectoryRead() attempts to read the next directory entry into the location specified by <em>dirent</em> from the file specified by <em>dirdescriptor</em>. The <em>dirdescriptor</em> should have been obtained by a previous call to VMDirectoryOpen(). When a thread calls

VMDirectoryRead() it blocks in the wait state VM_THREAD_STATE_WAITING if the reading of the directory entry cannot be completed immediately.

<h1>Return Value</h1>

Upon successful reading from the directory, VMDirectoryRead() returns

VM_STATUS_SUCCESS, upon failure VMDirectoryRead() returns

VM_STATUS_FAILURE. If <em>dirent</em> parameter is NULL, VMDirectoryRead() returns VM_STATUS_ERROR_INVALID_PARAMETER.







VMDirectoryRewind    Rewinds a previously opened directory to the beginning.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMDirectoryRewind(<strong>int</strong> dirdescriptor);

<h1>Description</h1>

VMDirectoryRewind() attempts to rewind the directory specified by <em>dirdescriptor</em> to the beginning. The <em>dirdescriptor</em> should have been obtained by a previous call to VMDirectoryOpen(). When a thread calls VMDirectoryRewind() it should block in the wait state VM_THREAD_STATE_WAITING if it is not possible to complete the rewind operation immediately.

<h1>Return Value</h1>

Upon successful rewinding of the directory, VMDirectoryRewind() returns

VM_STATUS_SUCCESS, upon failure VMDirectoryRewind() returns VM_STATUS_FAILURE.




VMDirectoryCurrent Fill the absolute path with the current working directory in the mounted FAT filesystem.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMDirectoryCurrent(<strong>char</strong> *abspath);

<h1>Description</h1>

VMDirectoryCurrent() attempts to place the absolute path of the current working directory in the location specified by <em>abspath</em>.

<h1>Return Value</h1>

Upon successful copying of the directory name into <em>abspath</em>, VMDirectoryCurrent() returns VM_STATUS_SUCCESS, if <em>abspath</em> is NULL then VMDirectoryCurrent() returns VM_STATUS_INVALID_PARAMETER.




VMDirectoryChange Changes the current working directory of the mounted FAT file system.

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMDirectoryChange(<strong>const</strong> <strong>char</strong> *path);

<h1>Description</h1>

VMDirectoryChange() attempts to change the current working directory of the mounted FAT file system to the name specified by <em>path</em>.

<h1>Return Value</h1>

Upon successful changing of the directory to <em>path</em>, VMDirectoryChange() returns

VM_STATUS_SUCCESS, if path specified by <em>path</em> does not exist,

VMDirectoryChange() returns VM_STATUS_FAILURE. If <em>path</em> is NULL then VMDirectoryChange() returns VM_STATUS_INVALID_PARAMETER.







VMDirectoryCreate – Attempts to create a directory in the mounted FAT file system.

(EXTRA CREDIT ONLY)

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMDirectoryCreate(<strong>const</strong> <strong>char</strong> *dirname);

<h1>Description</h1>

VMDirectoryCreate() attempts to create a directory in the mounted FAT file system specified by <em>dirname</em>.

<h1>Return Value</h1>

Upon successful creation of the directory, VMDirectoryCreate() returns

VM_STATUS_SUCCESS, if <em>dirname</em> is NULL then VMDirectoryCreate() returns VM_STATUS_INVALID_PARAMETER. If the directory is unable to be created, VMDirectoryCreate() returns VM_STATUS_FAILURE.







VMDirectoryUnlink – Removes a directory or file from the mounted FAT file system. (EXTRA CREDIT ONLY)

<h1>Synopsys</h1>

<strong>#include</strong> “VirtualMachine.h”




TVMStatus VMDirectoryUnlink(<strong>const</strong> <strong>char</strong> *path);

<h1>Description</h1>

VMDirectoryUnlink() attempts to remove the file or directory specified by <em>path</em> from the mounted FAT file system. VMDirectoryUnlink() will fail if the file or directory is current opened, or if the directory attempting to be unlinked is a parent of an open file or directory.

<h1>Return Value</h1>

Upon successful removing of the directory or file, VMDirectoryUnlink() returns

VM_STATUS_SUCCESS, upon failure VMDirectoryUnlink() returns VM_STATUS_FAILURE. If <em>path</em> is NULL then VMDirectoryUnlink() returns VM_STATUS_INVALID_PARAMETER.





