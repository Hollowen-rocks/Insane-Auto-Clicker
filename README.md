# InsaneAutoClicker

**InsaneAutoClicker** is a high-performance, multithreaded auto-clicker written in C#. This program simulates rapid mouse clicks with 10 parallel threads, achieving extremely high click rates. The auto-clicker runs until the F9 key is pressed, at which point it stops all click threads and terminates cleanly.

## Features

- Multithreaded Clicking: Utilizes 10 threads to maximize click performance.
- High Click Rate: Each thread simulates left mouse button clicks as fast as possible.
- Real-Time Click Rate Monitoring: Calculates and displays clicks per second (CPS) every second.
- Graceful Termination: Stops all click threads upon pressing the F9 key.
 
## How It Works

- The program starts 10 threads, each executing a loop that simulates a mouse click (left button down and up).
- The main thread monitors and calculates the click rate every second, displaying the CPS in the console.
- A separate thread monitors the F9 key press to stop the program.

## Requirements 

- **Windows OS:** This program uses `user32.dll` which is specific to Windows.
- **.NET Framework:** Make sure you have the .NET Framework installed on your system.



## Code Snippet
```csharp
using System;
using System.Diagnostics;
using System.Runtime.InteropServices;
using System.Threading;

class Program
{
    [DllImport("user32.dll", SetLastError = true)]
    static extern void mouse_event(uint dwFlags, uint dx, uint dy, uint dwData, UIntPtr dwExtraInfo);

    [DllImport("user32.dll")]
    static extern short GetAsyncKeyState(int vKey);

    const uint MOUSEEVENTF_LEFTDOWN = 0x0002;
    const uint MOUSEEVENTF_LEFTUP = 0x0004;
    const int VK_F9 = 0x78;

    static long totalClicks = 0;
    static bool stop = false;

    static void Main()
    {
        Thread[] threads = new Thread[10];
        for (int i = 0; i < 10; i++)
        {
            threads[i] = new Thread(ClickLoop);
            threads[i].Start();
        }

        Stopwatch stopwatch = new Stopwatch();
        stopwatch.Start();

        // Separate thread for checking the F9 key
        Thread keyCheckerThread = new Thread(CheckKeyPress);
        keyCheckerThread.Start();

        while (!stop)
        {
            Thread.Sleep(1000);
            long clicks = Interlocked.Read(ref totalClicks);
            double cps = clicks / (stopwatch.ElapsedMilliseconds / 1000.0);
            Console.WriteLine($"CPS: {cps}");
            Interlocked.Exchange(ref totalClicks, 0);
            stopwatch.Restart();
        }

        // Wait for all threads to finish
        foreach (var thread in threads)
        {
            thread.Join();
        }
    }

    static void ClickLoop()
    {
        while (!stop)
        {
            mouse_event(MOUSEEVENTF_LEFTDOWN, 0, 0, 0, UIntPtr.Zero);
            mouse_event(MOUSEEVENTF_LEFTUP, 0, 0, 0, UIntPtr.Zero);
            Interlocked.Increment(ref totalClicks);
        }
    }

    static void CheckKeyPress()
    {
        while (!stop)
        {
            if (GetAsyncKeyState(VK_F9) != 0)
            {
                stop = true;
            }
            Thread.Sleep(50);
        }
    }
}
