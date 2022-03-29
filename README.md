# Tetrix
The final work of the college computation course
     
     This is the first version.    
     // win32-1.cpp : 定义应用程序的入口点。
//

#include "stdafx.h"
#include "Tetris.h"

#define MAX_LOADSTRING 100
#define GRID 30           // 方块边长
#define nGameWidth 10     // 工作区宽
#define nGameHeight 20    // 工作区高
#define nWidth 16         // 窗口宽
#define nHeight 22        // 窗口高

// 全局变量:
HINSTANCE hInst;								// 当前实例
TCHAR szTitle[MAX_LOADSTRING];					// 标题栏文本
TCHAR szWindowClass[MAX_LOADSTRING];			// 主窗口类名

// 此代码模块中包含的函数的前向声明:
ATOM				MyRegisterClass(HINSTANCE hInstance);
BOOL				InitInstance(HINSTANCE, int);
LRESULT CALLBACK	WndProc(HWND, UINT, WPARAM, LPARAM);
INT_PTR CALLBACK	About(HWND, UINT, WPARAM, LPARAM);

int APIENTRY _tWinMain(HINSTANCE hInstance,
	HINSTANCE hPrevInstance,
	LPTSTR    lpCmdLine,
	int       nCmdShow)
{
	UNREFERENCED_PARAMETER(hPrevInstance);
	UNREFERENCED_PARAMETER(lpCmdLine);

	// TODO: 在此放置代码。
	MSG msg;
	HACCEL hAccelTable;

	// 初始化全局字符串
	LoadString(hInstance, IDS_APP_TITLE, szTitle, MAX_LOADSTRING);
	LoadString(hInstance, IDC_TETRIS, szWindowClass, MAX_LOADSTRING);
	MyRegisterClass(hInstance);

	// 执行应用程序初始化:
	if (!InitInstance (hInstance, nCmdShow))
	{
		return FALSE;
	}

	hAccelTable = LoadAccelerators(hInstance, MAKEINTRESOURCE(IDC_TETRIS));

	// 主消息循环:
	while (GetMessage(&msg, NULL, 0, 0))
	{
		if (!TranslateAccelerator(msg.hwnd, hAccelTable, &msg))
		{
			TranslateMessage(&msg);
			DispatchMessage(&msg);
		}
	}

	return (int) msg.wParam;
}



//
//  函数: MyRegisterClass()
//
//  目的: 注册窗口类。
//
//  注释:
//
//    仅当希望
//    此代码与添加到 Windows 95 中的“RegisterClassEx”
//    函数之前的 Win32 系统兼容时，才需要此函数及其用法。调用此函数十分重要，
//    这样应用程序就可以获得关联的
//    “格式正确的”小图标。
//
ATOM MyRegisterClass(HINSTANCE hInstance)
{
	WNDCLASSEX wcex;

	wcex.cbSize = sizeof(WNDCLASSEX);

	wcex.style			= CS_HREDRAW | CS_VREDRAW;
	wcex.lpfnWndProc	= WndProc;
	wcex.cbClsExtra		= 0;
	wcex.cbWndExtra		= 0;
	wcex.hInstance		= hInstance;
	wcex.hIcon			= LoadIcon(hInstance, MAKEINTRESOURCE(IDC_TETRIS));
	wcex.hCursor		= LoadCursor(NULL, IDC_ARROW);
	wcex.hbrBackground	= NULL;
	wcex.lpszMenuName	= MAKEINTRESOURCE(IDC_TETRIS);
	wcex.lpszClassName	= szWindowClass;
	wcex.hIconSm		= LoadIcon(wcex.hInstance, MAKEINTRESOURCE(IDI_SMALL));

	return RegisterClassEx(&wcex);
}

//
//   函数: InitInstance(HINSTANCE, int)
//
//   目的: 保存实例句柄并创建主窗口
//
//   注释:
//
//        在此函数中，我们在全局变量中保存实例句柄并
//        创建和显示主程序窗口。
//
BOOL InitInstance(HINSTANCE hInstance, int nCmdShow)
{
	HWND hWnd;

	hInst = hInstance; // 将实例句柄存储在全局变量中

	hWnd = CreateWindow(szWindowClass, TEXT("俄罗斯方块"), WS_SYSMENU,
		400, 100, GRID * nWidth, GRID * nHeight - 10, NULL, NULL, hInstance, NULL);

	if (!hWnd)
	{
		return FALSE;
	}

	ShowWindow(hWnd, nCmdShow);
	UpdateWindow(hWnd);

	return TRUE;
}

//
//  函数: WndProc(HWND, UINT, WPARAM, LPARAM)
//
//  目的: 处理主窗口的消息。
//
//  WM_COMMAND	- 处理应用程序菜单
//  WM_PAINT	- 绘制主窗口
//  WM_DESTROY	- 发送退出消息并返回
//
//
LRESULT CALLBACK WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
{
	int wmId, wmEvent;
	PAINTSTRUCT ps;
	HDC hdc;
	
	switch (message)
	{
	case WM_CREATE:
		// 设置初始方块的类型
		srand((unsigned)time(NULL));
		type = rand() % 7;
		nType = rand() % 7;

		// 设置方块的初始位置和计时器时间间隔
		/*方块的初始位置指的是bricks[type][state][0][0]的位置*/
		point.x = 4;
		point.y = -3;
		SetTimer(hWnd, 1, 500, NULL);		

		// 播放音乐
		PlaySound(_T("background.wav"), NULL, SND_FILENAME|SND_ASYNC|SND_LOOP);
		break;

	case WM_TIMER:
		// 若游戏已经结束或已暂停，不做任何操作
		if (gameOverFlag || suspendFlag)
			break;

		// 游戏还未结束，方块自动往下移动
		if (CanMoveDown()) 
		{
			MoveDown();
		}
		// 若当前方块已经触底
		else 
		{
			// 固定当前方块
			Fixing();

			// 消行
			DeleteLines();

			if (gameOverFlag)
			{
				MessageBox(hWnd, TEXT("游戏结束！"), TEXT("警告"), MB_OK);
			}
		}

		// 刷新屏幕
		InvalidateRect(hWnd, NULL, TRUE);

		break;

	case WM_KEYDOWN:
		// 若游戏已经结束，只有按“重新开始”键才有反应
		if (gameOverFlag) {
			if (wParam == 'S')
			{
				gameOverFlag = false;
				Restart();
				InvalidateRect(hWnd, NULL, TRUE);
			}
			break;
		}

		// 若游戏已经暂停，只有按“P”键才有反应
		if (suspendFlag) {
			if (wParam == 'P')
				suspendFlag = false;
			else
			{
				MessageBox(hWnd, TEXT("请先按P键取消暂停！"), TEXT("警告"), MB_OK);
			}
			InvalidateRect(hWnd, NULL, TRUE);
			break;
		}

		// 游戏未结束
		switch(wParam)
		{
		// 往左移动
		case VK_LEFT:
			if (CanMoveLeft()) 
			{
				MoveLeft();
			}

			InvalidateRect(hWnd, NULL, TRUE);
			break;

		// 往右移动
		case VK_RIGHT:
			if (CanMoveRight()) 
			{
				MoveRight();
			}
			InvalidateRect(hWnd, NULL, TRUE);
			break;

		// 旋转
		case VK_UP:
			Rotate();
			InvalidateRect(hWnd, NULL, TRUE);
			break;

		// 往下移动
		case VK_DOWN:
			if (CanMoveDown()) 
			{
				MoveDown();
			}
			// 若当前方块已经触底
			else 
			{
				// 固定当前方块
				Fixing();

				// 消行
				DeleteLines();

				if (gameOverFlag)
				{
					MessageBox(hWnd, TEXT("游戏结束！"), TEXT("警告"), MB_OK);
				}
			}

			// 刷新屏幕
			InvalidateRect(hWnd, NULL, TRUE);
			break;

		// 直接下落
		case VK_SPACE:
			// 下落
			DropDown();

			// 消行
			DeleteLines();

			if (gameOverFlag)
				MessageBox(hWnd, TEXT("游戏结束！"), TEXT("警告"), MB_OK);
			else
				InvalidateRect(hWnd, NULL, TRUE);
			break;

		// 暂停
		case 'P':
			suspendFlag = !suspendFlag;
			InvalidateRect(hWnd, NULL, TRUE);
			break;

		// 重新开始
		case 'S':
			Restart();
			InvalidateRect(hWnd, NULL, TRUE);
			break;

		// 瞄准器
		case 'C':
			targetFlag = !targetFlag;
			InvalidateRect(hWnd, NULL, TRUE);
			break;

		default:
			break;
		}

	case WM_PAINT:
		hdc = BeginPaint(hWnd, &ps);

		// 绘制屏幕
		TDrawScreen(hdc, hWnd);

		ReleaseDC(hWnd, hdc);
		EndPaint(hWnd, &ps);
		break;
	case WM_DESTROY:
		PostQuitMessage(0);
		break;
	default:
		return DefWindowProc(hWnd, message, wParam, lParam);
	}
	return 0;
}

// “关于”框的消息处理程序。
INT_PTR CALLBACK About(HWND hDlg, UINT message, WPARAM wParam, LPARAM lParam)
{
	UNREFERENCED_PARAMETER(lParam);
	switch (message)
	{
	case WM_INITDIALOG:
		return (INT_PTR)TRUE;

	case WM_COMMAND:
		if (LOWORD(wParam) == IDOK || LOWORD(wParam) == IDCANCEL)
		{
			EndDialog(hDlg, LOWORD(wParam));
			return (INT_PTR)TRUE;
		}
		break;
	}
	return (INT_PTR)FALSE;
}

// 判断当前方块是否可以向下移动
bool CanMoveDown() 
{
	//TODO
	int low = 0;
	for (int i = 0; i < 4; i++)
	{
		for (int j = 0; j < 4; j++)
		{
			if (bricks[type][state][i][j])/*找到砖块*/
			{
				/*考虑是否触底*/
				low = max(low, point.y+i);
				/*如果触底*/
				if (low >= 19)
					return false;

				/*考虑是否有其他砖块*/
				if (i != 3)
				{
					if (!bricks[type][state][i + 1][j])/*如果是最底层，下面没有方块*/
					{
						if (workRegion[point.y + i + 1][point.x + j])
							return false;
					}
				}
				else if(i==3)/*已经是最底层了*/
				{
					if (workRegion[point.y + i + 1][point.x + j])
						return false;
				}
			}
		}
	}

	return true;
}

// 判断当前方块是否可以向左移动
bool CanMoveLeft() 
{
	//TODO
	int left = 19;
	for (int i = 0; i < 4; i++)
	{
		for (int j = 0; j < 4; j++)
		{
			if (bricks[type][state][i][j])/*找到砖块*/
			{
				/*考虑是否触边*/
				left = min(left, point.x + j);
				/*如果触边*/
				if (!left)
					return false;

				/*考虑左边是否有其他砖块*/
				if (j)
				{
					if (!bricks[type][state][i][j - 1])/*如果是最左边*/
					{
						if (workRegion[point.y + i][point.x + j - 1])
							return false;
					}
				}
				else if (!j)/*已经是最左边了*/
				{
					if (workRegion[point.y + i + 1][point.x + j - 1])
						return false;
				}
			}
		}
	}

	return true;
}

// 判断当前方块是否可以向右移动
bool CanMoveRight() 
{
	//TODO
	int right = 0;
	for (int i = 0; i < 4; i++)
	{
		for (int j = 0; j < 4; j++)
		{
			if (bricks[type][state][i][j])/*找到砖块*/
			{
				/*考虑是否触底*/
				right = max(right, point.x + j);
				/*如果触底*/
				if (right >= 9)
					return false;

				/*考虑右边是否有其他砖块*/
				if (j!=3)
				{
					if (!bricks[type][state][i][j + 1])/*如果是最左边*/
					{
						if (workRegion[point.y + i][point.x + j + 1])
							return false;
					}
				}
				else if (j==3)/*已经是最右边了*/
				{
					if (workRegion[point.y + i + 1][point.x + j + 1])
						return false;
				}
			}
		}
	}

	return true;
}

// 当前方块向下移动
void MoveDown() 
{
	//TODO
	point.y++;
}

// 当前方块向左移动
void MoveLeft() 
{
	//TODO	
	point.x--;
	return;
}

// 当前方块向右移动
void MoveRight() 
{
	//TODO
	point.x++;
	return;
}

// 旋转当前方块
void Rotate() 
{
	//TODO
	bool flag = true;/*判断能否旋转*/

	int cstate = state > 2 ? 0: state+1;
	for (int i = 0; i < 4; i++)
	{
		for (int j = 0; j < 4; j++)
		{
			if (bricks[type][cstate][i][j] > 0)
			{
				if (point.x + j > 9 || point.x + j < 0 || point.y + i>19)
					flag = false;
			}
		}
	}

	if (flag)
	{
		if (state <= 2)
			state++;
		else
			state = 0;
	}
	
}

// 将当前方块直接下落到最下面的位置
void DropDown() 
{
	//TODO
	while (CanMoveDown())
		MoveDown();
	return;
}

// 将当前方块固定在当前位置
void Fixing() 
{
	for (int i = 0; i < 4; i++) 
	{
		for (int j = 0; j < 4; j++) 
		{
			if (bricks[type][state][i][j] > 0 && point.y + i >= 0)
				workRegion [point.y + i][point.x + j] = bricks[type][state][i][j];
		}
	}
	if (GameOver()) 
	{
		gameOverFlag = true;		
	}
	else 
	{
		point.x = 4;
		point.y = -3;
		type = nType;
		nType = rand() % 7;
		state = 0;
	}
}

// 消去一行
void DeleteOneLine(int number) 
{
	//TODO
	/*number指的是所要消去的行的位置*/
	for (int i = number;i > 0;i--)
	{
		for (int j = 0;j < 10;j++)
			workRegion[i][j] = workRegion[i - 1][j];
	}
	return;
}

// 消行操作并更新得分
void DeleteLines() 
{
	//TODO
	for (int i = 0;i < 20;i++)
	{
		int j = 0;
		for (;j < 10;j++)
		{
			if (!workRegion[i][j])
				break;
		}
		if (j == 10)/*没有break，说明这一行可以消去*/
		{
			DeleteOneLine(i);/*消行*/
			tScore += 200;/*更新得分*/
		}
	}
}

// 判断游戏结束
bool GameOver() 
{
	//TODO
	
	return 0;
}

// 计算瞄准器位置
void ComputeTarget()
{
	Point org = point;
	while(CanMoveDown())
	{
		MoveDown();
	}
	target = point;
	point = org;
}

// 重新开始
void Restart()
{
	//TODO
	/*工作区全部置0*/
	for (int i = 0;i < 20;i++)
	{
		for (int j = 0;j < 10;j++)
			workRegion[i][j] = 0;
	}

	/*分数清零*/
	tScore = 0;

	/*位置初始化*/
	point.x = 4;
	point.y = -3;

	type = nType;
	nType = rand() % 7;
	state = 0;
}

// 绘制屏幕
void TDrawScreen(HDC hdc, HWND hWnd)
{
	HDC dcMem = CreateCompatibleDC(hdc);
	HBITMAP bmpMem = CreateCompatibleBitmap(hdc, nWidth * GRID,nHeight * GRID); //将位图选择进内存DC 
	SelectObject(dcMem, bmpMem); 

	//用白色填充内存DC的客户区 
	HBRUSH hBrush = CreateSolidBrush(RGB(255, 255, 255)); 
	SelectObject(dcMem, hBrush); 
	Rectangle(dcMem, 0, 0, nWidth * GRID,nHeight * GRID); 

	// 加载背景图片
	HDC dcPic = CreateCompatibleDC(NULL);
	HBITMAP bmp = (HBITMAP)LoadImage(NULL, TEXT("background.bmp"), IMAGE_BITMAP, nWidth * GRID, nHeight * GRID, LR_LOADFROMFILE);
	SelectObject(dcPic, bmp);
	BitBlt(dcMem, 0, 0, nWidth * GRID, nHeight * GRID, dcPic, 0, 0, SRCCOPY);

	// 画边框
	HPEN hPen = CreatePen(PS_SOLID, 3, RGB(0, 0, 0));
	SelectObject(dcMem, hPen);
	MoveToEx(dcMem, nGameWidth * GRID, 0, NULL);
	LineTo(dcMem, nGameWidth * GRID, nGameHeight * GRID);
	MoveToEx(dcMem, 0, nGameHeight * GRID, NULL);
	LineTo(dcMem, nGameWidth * GRID, nGameHeight * GRID);

	// 输出游戏说明
	HFONT hFont = CreateFont(18, 10, 0, 0, 200, false, false, false, 
		ANSI_CHARSET, OUT_CHARACTER_PRECIS, 
		CLIP_CHARACTER_PRECIS, PROOF_QUALITY,
		FF_MODERN, (LPCWSTR)"Microsoft YaHei UI");
	SelectObject(dcMem, hFont);
	SetBkMode(dcMem, TRANSPARENT);

	TCHAR sscore[1024];
	int slen = wsprintf(sscore, L"当前得分: %d", tScore);
	TextOut(dcMem, (nGameWidth) * GRID + 5, 1 * GRID, TEXT("下一方块:"), 5);
	TextOut(dcMem, (nGameWidth) * GRID + 5, 6 * GRID, sscore, slen);
	TextOut(dcMem, (nGameWidth) * GRID + 5, 8 * GRID, TEXT("游戏说明:"), 5);
	TextOut(dcMem, (nGameWidth) * GRID + 25, 9 * GRID, TEXT("向左：←"), 4);
	TextOut(dcMem, (nGameWidth) * GRID + 25, 10 * GRID, TEXT("向右：→"), 4);
	TextOut(dcMem, (nGameWidth) * GRID + 25, 11 * GRID, TEXT("向下：↓"), 4);
	TextOut(dcMem, (nGameWidth) * GRID + 25, 12 * GRID, TEXT("旋转：↑"), 4);
	TextOut(dcMem, (nGameWidth) * GRID + 25, 13 * GRID, TEXT("直接下落：空格"), 7);
	TextOut(dcMem, (nGameWidth) * GRID + 25, 14 * GRID, TEXT("暂停/开始：P"), 7);
	TextOut(dcMem, (nGameWidth) * GRID + 25, 15 * GRID, TEXT("重新开始：S"), 6);
	TextOut(dcMem, (nGameWidth) * GRID + 25, 16 * GRID, TEXT("瞄准器：C"), 5);
	if(suspendFlag)
	{
		TextOut(dcMem, (nGameWidth) * GRID + 5, 18 * GRID, TEXT("已暂停..."), 6);
	}

	// 释放资源		
	DeleteObject(hPen);

	// 画工作区
	hPen = CreatePen(PS_SOLID, 3, RGB(0, 0, 0));
	SelectObject(dcMem, hPen);	
	for (int i = 0; i < 20; i++) 
	{
		for (int j = 0; j < 10; j++) 
		{
			if (workRegion [i][j] > 0) 
			{
				hBrush = CreateSolidBrush(BrickColor[workRegion [i][j] - 1]);
				SelectObject(dcMem, hBrush);

				// 画方格
				Rectangle(dcMem, j * GRID, i * GRID, (j + 1) * GRID, (i + 1) * GRID);

				DeleteObject(hBrush);				
			}
		}
	}

	// 画当前方块
	hBrush = CreateSolidBrush(BrickColor[type]);
	SelectObject(dcMem, hBrush);
	for (int i = 0; i < 4; i++) 
	{
		for (int j = 0; j < 4; j++) 
		{
			int value = bricks[type][state][i][j];
			if (value > 0) 
			{
				Rectangle(dcMem, (point.x + j) * GRID, (point.y + i) * GRID, (point.x + j + 1) * GRID, (point.y + i + 1) * GRID);
			}
		}
	}
	DeleteObject(hBrush);

	// 画下一方块
	hBrush = CreateSolidBrush(BrickColor[nType]);
	SelectObject(dcMem, hBrush);
	for (int i = 0; i < 4; i++) 
	{
		for (int j = 0; j < 4; j++) 
		{
			int value = bricks[nType][0][i][j];
			if (value > 0) 
			{
				Rectangle(dcMem, (10.5 + j) * GRID, (2 + i) * GRID, (10.5 + j + 1) * GRID, (2 + i + 1) * GRID);
			}
		}
	}
	DeleteObject(hBrush);

	// 画瞄准器
	if (targetFlag)
	{
		ComputeTarget();
		hBrush = CreateSolidBrush(RGB(255, 255, 255));
		SelectObject(dcMem, hBrush);
		for (int i = 0; i < 4; i++) 
		{
			for (int j = 0; j < 4; j++) 
			{
				int value = bricks[type][state][i][j];
				if (value > 0) 
				{
					Rectangle(dcMem, (target.x + j) * GRID, (target.y + i) * GRID, (target.x + j + 1) * GRID, (target.y + i + 1) * GRID);
				}
			}
		}
		DeleteObject(hBrush);
	}
	
	BitBlt(hdc, 0, 0, nWidth * GRID, nHeight * GRID, dcMem, 0, 0, SRCCOPY);

	DeleteObject(hPen);
	DeleteObject(hFont);
	DeleteObject(bmpMem);
	DeleteObject(bmp);
	ReleaseDC(hWnd, dcMem);
	ReleaseDC(hWnd, dcPic);
}
