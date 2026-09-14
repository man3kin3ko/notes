В Win32 (Windows API) графические приложения представляют собой бесконечный цикл, который обрабатывает события ОС (клики мыши, ввод клавиатуры). Этот цикл находится внутри функции WinMain (wWinMain):

```c++
int APIENTRY wWinMain (HINSTANCE hInstance, 
					HINSTANCE hPrevInstance, 
					PWSTR pCmdLine, 
					int nCmdShow)
{
	MSG msg;
	while (GetMessage(&msg, NULL, 0, 0)) { //получить событие ОС
	    TranslateMessage(&msg);
	    DispatchMessage(&msg); // передать в WNDPROC
	}
}
```

Входной точной для этих сообщений должна быть пользовательская функция с типом `WNDPROC` из `winuser.h`, называемая оконной процедурой.