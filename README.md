# 4d-plugin-migration-to-v17

#### callback system

It is no longer permissible to call ``PA_UnfreezeProcess`` from a non-4D execution context. 

**Plan**: Use simple global variable protected by ``std::lock_guard<std::mutex>`` (need to swich to C+11 and libc on Mac)

