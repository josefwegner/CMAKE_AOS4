# CMAKE_AOS4
## Why does this repository exist?
Because I have been fighting with CMAKE for long enough. It is about time to start embracing it and giving up some time to learn it since, in the long run, more time will be saved.

## Toolchain
There is no known CMAKE toolchain file for our cross-development when porting stuff to AmigaOS4. Due to this, using CMAKE instantly becomes a barrier. The purpose is to provide a basic toolchain file that can be used. It was provided to me by AFXGROUP and I have uploaded it into the same directory a this README file. It is probably not perfect, but I encourage (need) PRs.

## Why don't I like CMAKE?
Because I never saw an issue with tools such as AUTOMAKE etc and find them a lot easier to use rather than learning yet another way to do the same thing (re-inventing the wheel). In a lot of cases, it is as easy as running ./configure --build=BUILD_TRIPLET --host=HOST_TRIPLET [--target=TARGET_TRIPLET]. Sure, there there can be a lot of necessary addtions too, but they are all standard. If you want to disable something, then --enable-feature=OFF etc.

I also find it frustrating how every educational video you see of CMAKE is not based on the need to port things, though, it is probably my lack of motivation and/or searching. It seems to me that the assumption is always that you are generating a project for the same target as your build machine! This raises big issues when it comes to finding packages (see below).

## My findings regarding the seeking of packages
### pkg-config
It seems that CMAKE has different methods of retrieving packages. One of the common CMAKE modules is named pkg-config. Under the hood, it uses... pkg-config, of course. I have a gripe with pkg-config to begin with, since not every package is supplied with a .pc file - especially those packages uploaded to OS4Depot.net.

In any case, we must ensure that CMAKE/pkg-config knows not to search for .pc files belonging to the BUILD machine! We only want .pc files to be searched within the relevant SDK directory used on the BUILD machine. For instance, when you install a library from OS4Depot.net, it may very well comes with a .pc file. When you install the library into your BUILD machines SDK location, the library, its headers and the .pc file end up installed. Fine. As an example, you may left with something like:

/sdk/local/newlib/lib/libfoo.a
/sdk/local/newlib/lib/pkgconfig/foo.pc
/sdk/local/clib2/lib/libfoo.a
/sdk/local/clib2/lib/pkgconfig/foo.pc
/sdk/local/common/include/foo.h

Hopefully, the contents of the .pc file is not a hardcoded path to the library creator's/porter's location. This is another reason why it is useful to have a symbolic link /sdk/ that points to your actual SDK installation directory. On the AmigaOne machine, SDK: is a logical assign to the SDK folder and this is therefore analogous.

Anyway, issues arise when libfoo also exists on your BUILD machine. You might have something like this on a linux machine:

/usr/lib/libfoo.a
/usr/include/foo.h
/usr/share/pkgconfig/foo.pc

pkg-config will implicitly search in a list of default locations on your BUILD machine first which is not an abnormal thing for a program to do. There are also a number of suggestions that changing the environment variable PKG_CONFIG_PATH is all that is needed to get pkg-config to look in the correct locations. This is wrong. PKG_CONFIG_PATH is used in addition to the default locations, though, from the MAN page it does look like the paths listed in PKG_CONFIG_PATH are searched first. But, in order to eliminate the BUILD machine's files at all, it is best to set PKG_CONFIG_LIBDIR entirely. This will replace the default locations completely.

Therefore, when I use CMAKE for a project for which I want to cross-compile / port I will always set PKG_CONFIG_PATH to something like /sdk/local/newlib/lib/pkgconfig/

### find_module
The CMAKE pkg-config module is not the only way to find libraries though. Actually, from what I have read, I think it is discouraged since not everything comes with a .pc file.

### Finding / linking libraries when generating for AmigaOS4
Often, the CMakeLists.txt file needs to be hacked. Either there is no corresponding .pc file, so PKG_CHECK_MODULES cannot be used, or, there is no corresponding .cmake file for the find_module function. I believe there is no getting around this (let me know if there is an easier way; of course, one can write their own .pc / .cmake file!). Therefore, my general approach is to add something like:

```
if ( AMIGAOS4 )
  target_link_libraries(PROGRAM, "-lfoo")
else ()
  find_package(foo x.y REQUIRED)
endif ()
```
