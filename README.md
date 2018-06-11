# 4d-plugin-migration-to-v17

#### phase 1: callback system

done for [growl](https://github.com/miyako/4d-plugin-growl)

It is no longer permissible to call ``PA_UnfreezeProcess`` from a non-4D execution context. 

**Plan**: Use simple global variable protected by ``std::lock_guard<std::mutex>`` (need to swich to C+11 and libc on Mac)

* Model

```c
#include <mutex>

std::basic_string<PA_Unichar>methodName;
PA_long32 processNum = 0;

BOOL mySignal = FALSE;

void resume()
{	
	std::mutex m;
	std::lock_guard<std::mutex> lock(m);

	mySignal = TRUE;
}

void loop()
{
	PA_YieldAbsolute();
	
	while (!PA_IsProcessDying())
	{
		PA_YieldAbsolute();
		
		std::mutex m;
		std::lock_guard<std::mutex> lock(m);
		
		if(mySignal)
		{
			PA_Unistring methodName = PA_CreateUnistring((PA_Unichar *)method.c_str());
			PA_ExecuteMethod(&methodName);
			mySignal = FALSE;
		}else
		{
			PA_PutProcessToSleep(PA_GetCurrentProcessNumber(), 59);
		}
	}
	PA_KillProcess();

	std::mutex m;
	std::lock_guard<std::mutex> lock(m);
	
	processNum = 0;
}

void TEST(PA_PluginParameters params)
{
	if(!processNum)
	{
		std::mutex m;
		std::lock_guard<std::mutex> lock(m);
		
		PA_Unistring *param1 = PA_GetStringParameter(params, 1);
		method = (const PA_Unichar *)param1->fString;
		processNum = PA_NewProcess((void *)loop, (PA_long32)0, (PA_Unichar *)"$\0\0\0");
	}
	
	PA_RunInMainProcess((PA_RunInMainProcessProcPtr)resume, NULL);
}
```

#### phase 2: thread safety

done for [growl](https://github.com/miyako/4d-plugin-growl)

add ``"threadSafe": true`` to manifest if applicable

use native API instead of ``PA_ConvertCharsetToCharset`` which is thread unsafe

except for commands that call ``PA_RunInMainProcess`` which is unsafe

#### phase 3: object/collection support

use new entry points ``EX_SET_OBJ_VALUE`` and ``EX_GET_OBJ_VALUE``