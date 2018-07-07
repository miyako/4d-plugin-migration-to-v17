# 4d-plugin-migration-to-v17

#### phase 1: review callback system

[growl](https://github.com/miyako/4d-plugin-growl)
[folder-watch](https://github.com/miyako/4d-plugin-folder-watch)
[user-notification](https://github.com/miyako/4d-plugin-user-notification)
[apple-file-promises](https://github.com/miyako/4d-plugin-apple-file-promises)
[message-file-drop](https://github.com/miyako/4d-plugin-message-file-drop)*

``*`` need to review ``mutex`` code

#### phase 2: thread safety

done for 
[growl](https://github.com/miyako/4d-plugin-growl) 
[folder-watch](https://github.com/miyako/4d-plugin-folder-watch)† 
[uti-tools](https://github.com/miyako/4d-plugin-uti-tools) 
[curl-http](https://github.com/miyako/4d-plugin-curl-http)* 
[xlsxio](https://github.com/miyako/4d-plugin-xlsxio) [lha](https://github.com/miyako/4d-plugin-lha)
[user-notification](https://github.com/miyako/4d-plugin-user-notification) 
[pacparser](https://github.com/miyako/4d-plugin-pacparser)
[common-crypto](https://github.com/miyako/4d-plugin-common-crypto)
[snowball](https://github.com/miyako/4d-plugin-snowball)
[free-xl](https://github.com/miyako/4d-plugin-free-xl)
[document-properties](https://github.com/miyako/4d-plugin-document-properties)
[cpu](https://github.com/miyako/4d-plugin-cpu)
[script](https://github.com/miyako/4d-plugin-script)
[x-phonetic](https://github.com/miyako/4d-plugin-x-phonetic)
[text-convert](https://github.com/miyako/4d-plugin-text-convert)
[time-and-number](https://github.com/miyako/4d-plugin-time-and-number)†
[xls](https://github.com/miyako/4d-plugin-xls)

``*`` need to review blob code

``†`` need to review array code

* add ``"threadSafe": true`` to manifest if applicable

use native API instead of ``PA_ConvertCharsetToCharset`` which is thread unsafe

typical unsafe entry points are

``PA_RunInMainProcess``, ``PA_NewProcess`` (background processes in general)    
``PA_CreatePicture``, ``PA_CreateNativePictureForScreen`` (pictures in general)  
``PA_Set*InArray`` (arrays in general)  

change parameter types from array, ~~blob~~, picture to text if applicable

**UPDATE**: ``PA_NewHandle`` is now allowed in preemptive mode (no more ``-10530``, c.f. ``ACI0098388``)

``PA_GetBlobHandleParameter`` now works too; but not ``PA_GetBlobParameter``

* pictures are still not allowed

nor using ``EXECUTE METHOD BY ID`` to pass blob (unlike ``EXECUTE COMMAND``)

#### phase 3: object/collection support

use new entry points ``EX_SET_OBJ_VALUE`` and ``EX_GET_OBJ_VALUE``
