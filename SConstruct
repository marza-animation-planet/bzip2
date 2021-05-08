import sys
import excons
import SCons.Script # pylint: disable=import-error


env = excons.MakeBaseEnv()

staticlib = (excons.GetArgument("bz2-static", 1, int) != 0)
libsuffix = excons.GetArgument("bz2-suffix", "")

out_incdir = excons.OutputBaseDirectory() + "/include"
out_libdir = excons.OutputBaseDirectory() + "/lib"

def BZ2Name():
   name = "bz2" + libsuffix
   if sys.platform == "win32" and staticlib:
      # On windows, 'lib' is part of the library name for static lib
      # static: libbz2<suffix>.lib
      # shared: bz2<suffix>.lib + bz2<suffix>.dll
      name = "lib" + name
   return name

def BZ2Path():
   name = BZ2Name()
   if sys.platform == "win32":
      libname = name + ".lib"
   else:
      libname = "lib" + name + (".a" if staticlib else excons.SharedLibraryLinkExt())
   return out_libdir + "/" + libname

def RequireBZ2(env):
   if not staticlib and sys.platform == "win32":
      env.Append(CPPDEFINES=["BZ_DLL"])
   env.Append(CPPPATH=[out_incdir])
   env.Append(LIBPATH=[out_libdir])
   excons.Link(env, BZ2Path(), static=staticlib, force=True, silent=True)


defs = []
if not staticlib:
   defs.append("BZ_DLL")

cppflags = ""
if sys.platform != "win32":
   cppflags += " -Wno-unused-parameter"
else:
   # 4100: unreferenced formal parameter
   # 4127: conditional expression is constant
   # 4244: uint to uchar conversion
   # 4267: size_t to int32 conversion
   # 4702: unreachable code
   cppflags += " /wd4127 /wd4244 /wd4100 /wd4267 /wd4702"


prjs = [
   {  "name": BZ2Name(),
      "alias": "bz2",
      "type": "%slib" % ("static" if staticlib else "shared"),
      "defs": defs + (["BZ_DLL_EXPORTS"] if (sys.platform == "win32" and not staticlib) else []),
      "symvis": "default",
      "version": "1.0.8",
      "soname": "libbz2.so.1",
      "install_name": "libbz2.1.dylib",
      "cppflags": cppflags,
      "install": {"include": ["bzlib.h"]},
      "srcs": ["blocksort.c", "huffman.c", "crctable.c", "randtable.c", "compress.c", "decompress.c", "bzlib.c"]
   },
   {  "name": "bzip2",
      "type": "program",
      "alias": "bz2-tools",
      "cppflags": cppflags,
      "srcs": ["bzip2.c"],
      "custom": [RequireBZ2]
   },
   {  "name": "bzip2recover",
      "type": "program",
      "alias": "bz2-tools",
      "cppflags": cppflags,
      "srcs": ["bzip2recover.c"],
      "custom": [RequireBZ2]
   }
]

excons.AddHelpOptions(bz2="""BZIP2 OPTIONS
  bz2-static=0|1   : Build static or shared library. [1]
  bz2-suffix=<str> : Library name suffix.            []""")

excons.DeclareTargets(env, prjs)

SCons.Script.Export("BZ2Name BZ2Path RequireBZ2")
