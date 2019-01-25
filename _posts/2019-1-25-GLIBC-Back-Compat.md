---
layout: post
title: Make binaries portable across linux distros
---

When compiling a static binary on one linux distribution, it is not guaranteed to be compatible with another distribution out of the box. This is even true, when compiling statically. For example when I cross-compile monerod for aarch64 on an x86_64 machine running ubuntu 18.04, I get the following error when running the binary on my debian 9 aarch64 system:

    ./monerod: /lib/aarch64-linux-gnu/libc.so.6: version `GLIBC_2.27' not found (required by ./monerod)
    ./monerod: /lib/aarch64-linux-gnu/libc.so.6: version `GLIBC_2.25' not found (required by ./monerod)

The system c-library glibc is evidently not statically linked, even though the binary is compiled with `-static`. `gcc` does not link glibc statically, and trying to override this is heavily discouraged. When running a binary on linux, it either looks for the host system's libc, or if specified in the symbol table, a locally linked version shipped with the main executable. Newer versions of glibc are not backwards compatible to older version. Compiling the binary on our ubuntu system we used a different version of the glibc library compared to the glibc version we have installed on our debian 9 system. The newer library on our ubuntu machine contains improvements and security fixes that the older library on debian does not know about yet. This leaves us with undefined, or differently defined symbols in our static binary. Luckily glibc versions its symbols with semver and allows selecting older versions of system functions. 

There are two different ways to ship around glibc versioning and still be able to run a static binary compiled on one distribution on another. We now chieck which two symbols create the error message above. For this first run `objdump -T monerod | grep 2\\.27`:

    0000000000000000      DF *UND*  0000000000000000  GLIBC_2.27  glob

The conflicting symbol is `glob`, which actually received a security update after a [CVE](https://www.cvedetails.com/cve/CVE-2017-15804/) was found. We then check what the semver of the `glob` in glibc on the debian system is with `objdump -T /lib/aarch64-linux-gnu/libc.so.6 | grep glob`:

    000000000009dc28 g    DF .text  0000000000001508  GLIBC_2.17  glob
    
Lets now create a new `*.c/*.cpp` file that we compile and link into our final executable. Here we redefine glob and force libc to link with the older version instead. I've already added the latest sane versions we can select for other architectures besides aarch64 as well. 

    #include <cstddef>
    #include <cstdint>
    #include <strings.h>
    #include <string.h>
    #include <glob.h>
    #include <unistd.h>
    #include <fnmatch.h>
    
    #if defined(HAVE_SYS_SELECT_H)
    #include <sys/select.h>
    #endif
    #undef glob
    extern "C" int glob_old(const char * pattern, int flags, int (*errfunc) (const char *epath, int eerrno), glob_t *pglob);
    #ifdef __i386__
        __asm__(".symver glob_old,glob@GLIBC_2.1");
    #elif defined(__amd64__)
        __asm__(".symver glob_old,glob@GLIBC_2.2.5");
    #elif defined(__arm__)
        __asm(".symver glob_old,glob@GLIBC_2.4");
    #elif defined(__aarch64__)
        __asm__(".symver glob_old,glob@GLIBC_2.17");
    #endif

    extern "C" int __wrap_glob(const char * pattern, int flags, int (*errfunc) (const char *epath, int eerrno), glob_t *pglob)
    {
        return glob_old(pattern, flags, errfunc, pglob);
    }
    
Lastly we need to make sure that this newly defined glob is wrapped  in the old function by adding `-Wl,--wrap=glob` to the linker flags. 

Now lets run `objdump -T monerod | grep 2\\.27`:

    0000000000000000      DF *UND*  0000000000000000  GLIBC_2.25  __explicit_bzero_chk

The conflicting flag here is `__explicit_bzero_chk`. This function is not complex, it just checks if a memory region is zeroed. Instead of selecting a different version, we just redefine the function and add an alias to use it when library internal calls to it are made. The following is appended to the previously generated file:

    /* glibc-internal users use __explicit_bzero_chk, and explicit_bzero
    redirects to that.  */
    #undef explicit_bzero
    /* Set LEN bytes of S to 0.  The compiler will not delete a call to
    this function, even if S is dead after the call.  */
    void explicit_bzero (void *s, size_t len)
    {
        memset (s, '\0', len);
        /* Compiler barrier.  */
        asm volatile ("" ::: "memory");
    }

    // Redefine explicit_bzero_chk
    void __explicit_bzero_chk (void *dst, size_t len, size_t dstlen)
    {
        /* Inline __memset_chk to avoid a PLT reference to __memset_chk.  */
        if (__glibc_unlikely (dstlen < len))
            __chk_fail ();
        memset (dst, '\0', len);
        /* Compiler barrier.  */
        asm volatile ("" ::: "memory");
    }
    /* libc-internal references use the hidden
    __explicit_bzero_chk_internal symbol.  This is necessary if
    __explicit_bzero_chk is implemented as an IFUNC because some
    targets do not support hidden references to IFUNC symbols.  */
    #define strong_alias (__explicit_bzero_chk, __explicit_bzero_chk_internal)

This method has the benefit that no additional compiler flags have to be added and that once defined, we do not have to make sure that we select the correct glibc versions for different distros/architectures as we had to for `glob`. If we now re-compile and run `objdump -p monerod` we'll see that none of the symbols versioned with `2.25` or `2.27` remain. In the specific case of monerod, the binary is now compatible across all systems with an installation of glibc version 2.17 or higher.


