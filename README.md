# musl-hack-dns Alpine image

## MUSL library kube DNS fix
- https://git.musl-libc.org/cgit/musl/
- https://github.com/kubernetes/kubernetes/issues/56903

Modify function `name_from_dns` from the file `/musl/src/network/lookup_name.c` in git repo : https://git.musl-libc.org/cgit/musl/ with the following hack fix :

```c
    //musl/src/network/lookup_name.c, function 'name_from_dns'
    static const struct { int af; int rr; } afrr[2] = { 
        { .af = AF_INET6, .rr = RR_A },
        { .af = AF_INET, .rr = RR_AAAA },
    };  

    for (i=0; i<2; i++) {
        if (family != afrr[i].af) {
            qlens[nq] = __res_mkquery(0, name, 1, afrr[i].rr,
                0, 0, 0, qbuf[nq], sizeof *qbuf);
            if (qlens[nq] == -1) 
                return EAI_NONAME;
            nq++;
        }   

        //hack: if set the AF_UNSPEC family, just return ipv4 result
        if (family == AF_UNSPEC) break;
    }
 ```
 
 once done, you can compile the code with `gcc` docker container :
 
Run `gcc` container :
```
docker run --entrypoint=/bin/bash --rm -it --privileged gcc
```

Clone the `musl` library repository :
```
git clone git://git.musl-libc.org/musl
```

Edit `/musl/src/network/lookup_name.c` and modify the file according to the hack.
Compile the code :
```
./configure && make install
```
Copy the library to your host :
```
docker cp 0b6a77f89416:/usr/local/musl/lib/libc.so .
```

Then you can inject it in another container alpine :
```
kubectl cp libc.so namespace/pod-name:/mnt
```

Once copied, move the code to the appropriate location :
```
kubectl exec -it pod-name -- sh
cp /mnt/libc.so /lib/ld-musl-x86_64.so.1
```
