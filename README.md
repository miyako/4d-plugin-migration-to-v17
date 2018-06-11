# 4d-plugin-migration-to-v17

#### callback system

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
	while (!PA_IsProcessDying())
	{
		PA_PutProcessToSleep(PA_GetCurrentProcessNumber(), 59);

		std::mutex m;
		std::lock_guard<std::mutex> lock(m);
		
		if(mySignal)
		{
			PA_Unistring methodName = PA_CreateUnistring((PA_Unichar *)method.c_str());
			PA_ExecuteMethod(&methodName);
			mySignal = FALSE;
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
		processNum = PA_NewProcess((void *)loop, (PA_long32)0, (PA_Unichar *)"\0$\0\0\0");
	}
	
	PA_RunInMainProcess((PA_RunInMainProcessProcPtr)resume, NULL);
}
```
