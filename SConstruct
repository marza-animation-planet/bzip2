import sys
import excons


env = excons.MakeBaseEnv()

staticlib = (excons.GetArgument("bzip2-static", 1, int) != 0)
libsuffix = excons.GetArgument("bzip2-lib-suffix", "_s" if staticlib else "")

out_incdir = excons.OutputBaseDirectory() + "/include"
out_libdir = excons.OutputBaseDirectory() + "/lib"

def Bzip2Name():
   return ("lib" if sys.platform == "win32" else "") + "bz2" + libsuffix

def Bzip2Path():
   name = Bzip2Name()
   if sys.platform == "win32":
      libname = name + ".lib"
   else:
      libname = "lib" + name + ".a"
   return out_libdir + "/" + libname

def RequireBzip2(env):
   if not staticlib:
      env.Append(CPPDEFINES=["BZ_DLL"])
   env.Append(CPPPATH=[out_incdir])
   env.Append(LIBPATH=[out_libdir])
   excons.Link(env, Bzip2Name(), static=staticlib, force=True, silent=True)


defs = ["_FILE_OFFSET_BITS=64"]
if not staticlib:
   defs.append("BZ_DLL")
if sys.platform == "win32":
   defs.append("_CRT_SECURE_NO_WARNINGS")

cppflags = ""
if sys.platform != "win32":
   cppflags += " -Wno-unused-parameter"
else:
   # 4100: unreferenced formal parameter
   # 4127: conditional expression is constant
   # 4244: uint to uchar conversion
   # 4267: size_t to int32 conversion
   # 4996: POSIX name deprecated
   # 4702: unreachable code
   cppflags += " /wd4127 /wd4244 /wd4100 /wd4267 /wd4996 /wd4702"

libname = Bzip2Name()

prjs = [
   {  "name": libname,
      "type": "%slib" % ("static" if staticlib else "shared"),
      "defs": defs + ["BZ_DLL_EXPORTS"],
      "symvis": "default",
      "version": "1.0.6",
      "soname": "libbz2.so.1",
      "install_name": "libbz2.1.dylib",
      "cppflags": cppflags,
      "install": {"include": ["bzlib.h"]},
      "srcs": ["blocksort.c", "huffman.c", "crctable.c", "randtable.c", "compress.c", "decompress.c", "bzlib.c"]
   },
   {  "name": "bzip2",
      "type": "program",
      "defs": defs,
      "cppflags": cppflags,
      "srcs": ["bzip2.c"],
      "deps": [libname],
      "custom": [RequireBzip2]
   },
   {  "name": "bzip2recover",
      "type": "program",
      "defs": defs,
      "cppflags": cppflags,
      "srcs": ["bzip2recover.c"],
      "deps": [libname],
      "custom": [RequireBzip2]
   }
]

excons.DeclareTargets(env, prjs)

Export("Bzip2Name Bzip2Path RequireBzip2")
