

Most developers realize that PDB files are something that help you debug, but that's about it. Don't feel bad if you don't know what's going on with PDB files because while there is documentation out there, it's scattered around and much of it is for compiler and debugger writers. While it's extremely cool and interesting to write compilers and debuggers, that's probably not your job.

What I want to do here is to put in one place what everyone doing development on a Microsoft operating system has to know when it comes to PDB files. This information also applies to both native and managed developers, though I will mention a trick specific to managed developers. I'll start by talking about PDB file storage as well as the contents. Since the debugger uses the PDB files, I'll discuss exactly how the debugger finds the right PDB file for your binary. Finally, I'll talk about how the debugger looks for the source files when debugging and show you a favorite trick related to how the debugger finds source code.

Before we jump in, I need to define two important terms. A build you do on your development machine is a private build. A build done on a build machine is a public build. This is an important distinction because debugging binaries you build locally is easy, it is always the public builds that cause problems.

The most important thing all developers need to know: PDB files are as important as source code! Yes, that's red and bold on purpose. I've been to countless companies to help them debug those bugs costing hundreds of thousands of dollars and nobody can find the PDB files for the build running on a production server. Without the matching PDB files you just made your debugging challenge nearly impossible. With a huge amount of effort, my fellow Wintellectuals and I can find the problems without the right PDB files, but it will save you a lot of money if you have the right PDB files in the first place.

As John Cunningham, the development manager for all things diagnostics on Visual Studio, said at the 2008 PDC, "Love, hold, and protect your PDBs." At a minimum, every development shop must set up a Symbol Server. I've written about Symbol Servers in MSDN Magazine and more extensively in my book, Debugging .NET 2.0 Applications. You can also read the Symbol Server documentation itself in the Debugging Tools for Windows help file. Look at those resources to learn more about the details. Briefly, a Symbol Server stores the PDBs and binaries for all your public builds. That way no matter what build someone reports a crash or problem, you have the exact matching PDB file for that public build the debugger can access. Both Visual Studio and WinDBG know how to access Symbol Servers and if the binary is from a public build, the debugger will get the matching PDB file automatically.

Most of you reading this will also need to do one preparatory step before putting your PDB files in the Symbol Server. That step is to run the Source Server tools across your public PDB files, which is called source indexing. The indexing embeds the version control commands to pull the exact source file used in that particular public build. Thus, when you are debugging that public build you never have to worry about finding the source file for that build. If you're a one or two person team, you can sometimes live without the Source Server step. For the rest of you, read my article in MSDN Magazine on Source Server to learn how to use it.

The rest of this entry will assume you have set up Symbol Server and Source Server indexing. One good piece of news for those of you who will be using TFS 2010, out of the box the Build server will have the build task for Source Indexing and Symbol Server copying as part of your build.

One complaint I've heard against setting up a Symbol Server from some teams is that their software is too big and complex. I have to admit that when I hear people say that it translates to me as "My team is dysfunctional." There's no way your software is bigger and more complex than everything Microsoft does. They source index and store every single build of all products they ship into a Symbol Server. That means everything from Windows, to Office, to SQL, to Games and everything in between is stored in one central location. My guess is that Building 34 in Redmond is nothing but SAN drives to hold all of those files and everyone in that building is there to support those SANs. It's so amazing to be able to debug anything inside Microsoft and you never have to worry about symbols or source (provided you have appropriate rights to that source tree).

With the key infrastructure discussion out of the way, let me turn to what's in a PDB and how the debugger finds them. The actual file format of a PDB file is a closely guarded secret but Microsoft provides APIs to return the data for debuggers. A native C++ PDB file contains quite a bit of information:

    Public, private, and static function addresses
    Global variable names and addresses
    Parameter and local variable names and offsets where to find them on the stack
    Type data consisting of class, structure, and data definitions
    Frame Pointer Omission (FPO) data, which is the key to native stack walking on x86
    Source file names and their lines 

A .NET PDB only contains two pieces of information, the source file names and their lines and the local variable names. All the other information is already in the .NET metadata so there is no need to duplicate the same information in a PDB file.

When you load a module into the process address space, the debugger uses two pieces of information to find the matching PDB file. The first is obviously the name of the file. If you load ZZZ.DLL, the debugger looks for ZZZ.PDB. The extremely important part is how the debugger knows this is the exact matching PDB file for this binary. That's done through a GUID that's embedded in both the PDB file and the binary. If the GUID does not match, you certainly won't debug the module at the source code level.

The .NET compiler, and for native the linker, puts this GUID into the binary and PDB. Since the act of compiling creates this GUID, stop and think about this for a moment. If you have yesterday's build and did not save the PDB file will you ever be able to debug the binary again? No! This is why it is so critical to save your PDB files for every build. Because I know you're thinking it, I'll go ahead and answer the question already forming in your mind: no, there's no way to change the GUID.

However, you can look at the GUID value in your binary. Using a command line tool that comes with Visual Studio, DUMPBIN, you can list all the pieces of your Portable Executable (PE) files. To run DUMPBIN, open the Visual Studio 2008 Command Prompt from the Program's menu, as you will need the PATH environment variable set in order to find the DUMPBIN EXE. By the way, if you're interested in more about the information that DUMPBIN shows you, I highly recommend the definitive articles on the PE file by Matt Pietrek in the February 2002 and March 2002 issues of MSDN Magazine.

There are numerous command line options to DUMPBIN, but the one that shows us the build GUID is /HEADERS. The Pietrek articles will explain the output, but the important piece to us is the Debug Directories output:

Debug Directories
Time Type Size RVA Pointer
-------- ------ -------- -------- --------
4A03CA66 cv 4A 000025C4 7C4 Format: RSDS,
  {4B46C704-B6DE-44B2-B8F5-A200A7E541B0}, 1,
C:\junk\stuff\HelloWorld\obj\Debug\HelloWorld.pdb

With the knowledge of how the debugger determines the correctly matching PDB file, I want to talk about where the debugger looks for the PDB files. You can see all of this order loading yourself by looking at the Visual Studio Modules window, Symbol File column when debugging. The first place searched is the directory where the binary was loaded. If the PDB file is not there, the second place the debugger looks is the hard coded build directory embedded in the Debug Directories in the PE file. If you look at the above output, you see the full path C:\JUNK\STUFF\HELLOWORLD\OBJ\DEBUG\HELLOWORD.PDB. (The MSBUILD tasks for building .NET applications actually build to the OBJ\<CONFIG> directory and copy the output to DEBUG or RELEASE directory only on a successful build.) If the PDB file is not in the first two locations, and a Symbol Server is set up for the on the machine, the debugger looks in the Symbol Server cache directory. Finally, if the debugger does not find the PDB file in the Symbol Server cache directory, it looks in the Symbol Server itself. This search order is why your local builds and public build parts never conflict.

How the debugger searches for PDB files works just fine for nearly all the applications you'll develop. Where PDB file loading gets a little more interesting are those .NET applications that require you to put assemblies in the Global Assembly Cache (GAC). I'm specifically looking at you SharePoint and the cruelty you inflict on web parts, but there are others. For private builds on your local machine, life is easy because the debugger will find the PDB file in the build directory as I described above. The pain starts when you need to debug or test a private build on another machine.

On the other machine, what I've seen numerous developers do after using GACUTIL to put the assembly into the GAC is to open up a command window and dig around in C:\WINDOWS\ASSEMBLY\ to look for the physical location of the assembly on disk. While it is subject to change in the future, an assembly compiled for Any CPU is actually in a directory like the following:

C:\Windows\assembly\GAC_MSIL\Example\1.0.0.0__682bc775ff82796a

Example is the name of the assembly, 1.0.0.0 is the version number, and 682bc775ff82796a is the public key token value. Once you've deduced the actual directory, you can copy the PDB file to that directory and the debugger will load it.

If you're feeling a little queasy right now about digging through the GAC like this, you should, as it is unsupported and fragile. There's a better way that seems like almost no one knows about, DEVPATH. The idea is that you can set a couple of settings in .NET and it will add a directory you specify to the GAC so you just need to toss the assembly and it's PDB file into that directory so debugging is far easier. Only set up DEVPATH on development machines because any files stored in the specified directory are not version checked as they are in the real GAC.

By the way, if you search for DEVPATH in any internet search engine one of the top entries is an out of date blog entry by Suzanne Cook saying Microsoft was getting rid of DEVPATH. That is no longer true. As with any blog entry, look at the date on Suzanne's blog: 2003. That's the equivalent of 1670 in internet years.

To use DEVPATH, you will first create a directory that has read access rights for all accounts and at least write access for your development account. This directory can be anywhere on the machine. The second step is to set a system wide environment variable, DEVPATH whose value is the directory you created. The documentation on DEVPATH doesn't make this clear, but set the DEVPATH environment variable before you do the next step.

To tell the .NET runtime that you have DEVPATH set up requires you to add the following to your APP.CONFIG, WEB.CONFIG, or MACHINE.CONFIG as appropriate for your application:

<configuration>
   <runtime>
      <developmentMode developerInstallation="true"/>
   </runtime>
</configuration>

Once you turn on development mode, you'll know there's a problem with either the DEVPATH environment variable missing for the process or the path you set does not exist if your application dies at startup with a COMException with the error message saying the completely non-intuitive: "Invalid value for registry." Also, be extremely vigilant if you do want to use DEVPATH in MACHINE.CONFIG because every process on the machine is affected. Causing all .NET applications to fail on a machine won't win you many friends around the office.

The final item every developer needs to know about PDB files is how the source file information is stored in a PDB file. For public builds that have had source indexing tools run on them, the storage is the version control command to get that source file into the source cache you set. For private builds, what's stored is the full path to the source files that compiler used to make the binary. In other words, if you use a source file MYCODE.CPP in C:\FOO, what's embedded in the PDB file is C:\FOO\MYCODE.CPP. This is probably what you already suspected, but I just wanted to make it clear.

Ideally, all public builds are automatically being source indexed immediately and stored in your Symbol Server so if you don't have to even think any more about where the source code is. However, some teams don't do the source indexing across the PDB files until they have done smoke tests or other blessings to see if the build is good enough for others to use. That's a perfectly reasonable approach, but if you do have to debug the build before its source indexed, you had better pull that source code to the exact same drive and directory structure the build machine used or you may have some trouble debugging at the source code level. While both the Visual Studio debugger and WinDBG have options for setting the source search directories, I've found it hard to get right.

For smaller projects, it's no problem because there's always plenty of room for your source code. Where life is more difficult is on bigger projects. What are you going to do if you have 30 MB of source code and you have only 20 MB of disk space left on your C: drive? Wouldn't it be nice to have a way to control the path stored in the PDB file?

While we can't edit the PDB files, there's an easy trick to controlling the paths put inside the PDB files: SUBST.EXE. What SUBST does is associate a path with a drive letter. If you pull your source code down to C:\DEV and you execute "SUBST R: C:\DEV" the R: drive will now show at its top level the same files and directories if you typed "DIR C:\DEV." You'll also see the R: drive in Explorer as a new drive. You can also achieve the drive to path affect by mapping a drive to a shared directory in Explorer. I personally prefer the SUBST approach because it doesn't require any shares on the machine. While some of you are thinking that you can share through <DRIVE>$, some organizations disable that functionality.

What you'll do on the build machine is set a startup item that executes your particular SUBST command. When the build system account logs in, it will have the new drive letter available and that's where you'll do your builds. With complete control over the drive and root embedded in the PDB file, all you need to do to set up the source code on a test machine is to pull it down wherever you want and do a SUBST execution using the same drive letter the build machine used. Now there's no more thinking about source matching again in the debugger.

While not all of this information about PDB files I've discussed in this entry is entirely new, I didn't see it in one place before. I hope by getting it all together that you'll find it easier to deal with what's going on and debug your applications faster. Debugging faster means shipping faster so that's always high on the good things scale. Please ask any questions you may have on PDB files in comments, and I'll be happy to dig up the answers for you.
