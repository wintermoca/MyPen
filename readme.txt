##Winui3 Win2D를 이용한 PEN제작

#1 xaml파일 수정하기
기존의 mybutton과 관련된 항목을 제거하고
Nuget으로 다운로드한 CanvasControl class를 사용하기 위해 
 xmlns:canvas="using:Microsoft.Graphics.Canvas.UI.Xaml"
를 추가해 준다.

또한 Grid 레이아웃 패널을 활용하여 
PointerPressed
PointerReleased
PointerMoved
Draw
의 이벤트 처리기를 추가해 주고
ClearColor로 배경색을 설정해 줍니다.

```
<Grid>
    <canvas:CanvasControl
        PointerReleased="CanvasControl_PointerReleased"
        PointerPressed="CanvasControl_PointerPressed"
        PointerMoved="CanvasControl_PointerMoved"
        
        Draw="CanvasControl_Draw"
        
        ClearColor="NavajoWhite"
        />

</Grid>
```

#2 xaml.h 파일 수정
```
#include <winrt/Microsoft.Graphics.Canvas.UI.Xaml.h>
#include <winrt/Microsoft.UI.Input.h>
#include <winrt/Microsoft.UI.Xaml.Input.h>

using namespace winrt::Microsoft::UI;
using namespace winrt::Microsoft::Graphics::Canvas::UI::Xaml;
```
위의 헤더파일과 네임 스페이스를 추가하여 줍니다.

 bool flag;
 float px, py;
 std::vector<float> vx;
 std::vector<float> vy;
위의 변수를 해더파일에 추가하여 전역변수로서 사용될 수 있게끔 합니다.

#3 xaml.cpp 파일 수정

flag = false;
px = py = 100;
전역변수들을 초기화 하여주고

```
void winrt::MyPen::implementation::MainWindow::CanvasControl_PointerReleased(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e)
{
    flag = false;
    px = py = 0.0;
    vx.push_back(px);
    vy.push_back(py);
}


void winrt::MyPen::implementation::MainWindow::CanvasControl_PointerPressed(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e)
{
    flag = true;
}


void winrt::MyPen::implementation::MainWindow::CanvasControl_PointerMoved(winrt::Windows::Foundation::IInspectable const& sender, winrt::Microsoft::UI::Xaml::Input::PointerRoutedEventArgs const& e)
{
    CanvasControl canvas = sender.as<CanvasControl>();
    px = e.GetCurrentPoint(canvas).Position().X;
    py = e.GetCurrentPoint(canvas).Position().Y;
    if (flag) {
        vx.push_back(px);
        vy.push_back(py);
        canvas.Invalidate();


    }

}
```
위와같이 이벤트 함수들에게 기능을 부여합니다.
PointerPressed에는 마우스 입력을 감지하는 이벤트가 발생할 경우 flag의 값을 true로 바꿔주도록,
PointerReleased에는 마우스 입력종료를 감지하는 이벤트가 발생할 경우 flag는 false값으로 초기화 하고, vector뒤에 값을 넣어주도록 합니다.
PointerMoved에는 마우스 포인터가 움직이는동안 캔버스의 X,Y좌표의 위치를 px, py에 넣고, vx와 vy에 그 값을 집어넣어주게끔 합니다.
또한 canvas에 그려진것을 다시 그리기 위해 Invalidate함수를 사용합니다.

```
void winrt::MyPen::implementation::MainWindow::CanvasControl_Draw(winrt::Microsoft::Graphics::Canvas::UI::Xaml::CanvasControl const& sender, winrt::Microsoft::Graphics::Canvas::UI::Xaml::CanvasDrawEventArgs const& args)
{
    CanvasControl canvas = sender.as<CanvasControl>();
    int n = vx.size();
    for (int i = 1; i < n; i++) {
        if (vx[i] == 0 && vy[i] == 0) {
            i++; continue;
        }
        args.DrawingSession().DrawLine(vx[i - 1], vy[i - 1], vx[i], vy[i], Colors::DeepSkyBlue(), 16);
        args.DrawingSession().FillEllipse(vx[i - 1], vy[i - 1], 8, 8, Colors::DeepSkyBlue());
        args.DrawingSession().FillEllipse(vx[i], vy[i], 8, 8, Colors::DeepSkyBlue());

    }
}
```
마지막으로 Draw이벤트입니다.
vx에 담겨있는 크기만큼 차례대로 확인하여 0이 아닌 값이 있다면 DrawLine함수와 FillEllipse함수를 씁니다.




