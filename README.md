## WinAppDriver demo

https://github.com/Microsoft/WinAppDriver

Windows 10 must be set in 'Developer mode' - otherwise WinAppDriver.exe will not run!
(Windows 10) Settings > Update & Security > For developers

Installing and Running Windows Application Driver

    Download Windows Application Driver installer from https://github.com/Microsoft/WinAppDriver/releases
    Run the installer on a Windows 10 machine where your application under test is installed and will be tested
    Run WinAppDriver.exe from the installation directory (E.g. C:\Program Files (x86)\Windows Application Driver)

Windows Application Driver will then be running on the test machine listening to requests on the default IP address and port (127.0.0.1:4723). You can then run any of our Tests or Samples. WinAppDriver.exe can be configured to listen to a different IP address and port as follows:

WinAppDriver.exe 4727
WinAppDriver.exe 10.0.0.10 4725
WinAppDriver.exe 10.0.0.10 4723/wd/hub

    Note: You must run WinAppDriver.exe as administrator to listen to a different IP address and port.

	
Maven dependencies :

```
 <dependencies>
        <dependency>
            <groupId>org.seleniumhq.selenium</groupId>
            <artifactId>selenium-java</artifactId>
            <version>3.3.1</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
        </dependency>

        <dependency>
            <groupId>io.appium</groupId>
            <artifactId>java-client</artifactId>
            <version>5.0.0-BETA6</version>
        </dependency>
    </dependencies>
```

# Connecting to an existing window

Sample code is in C#, but illustrates what to do
```
// Start up WinAppDriver in 'root' mode to access the entrie Windows desktop
DesiredCapabilities appCapabilities = new DesiredCapabilities();
appCapabilities.SetCapability("app", "Root");
DesktopSession = new WindowsDriver<WindowsElement>(new Uri("http://127.0.0.1:4723"), appCapabilities);

// Launch your application (perhaps even by sending keys through the driver) then use the 'root' session to get the window handle
// In this example, the application is Cortana
DesktopSession.Keyboard.SendKeys(Keys.Meta + "s" + Keys.Meta);

var CortanaWindow = DesktopSession.FindElementByName("Cortana");
var CortanaTopLevelWindowHandle = CortanaWindow.GetAttribute("NativeWindowHandle");
CortanaTopLevelWindowHandle = (int.Parse(CortanaTopLevelWindowHandle)).ToString("x"); // Convert to Hex

// Create session by attaching to Cortana top level window
DesiredCapabilities appCapabilities = new DesiredCapabilities();
appCapabilities.SetCapability("appTopLevelWindow", CortanaTopLevelWindowHandle);
CortanaSession = new WindowsDriver<WindowsElement>(new Uri(WindowsApplicationDriverUrl), appCapabilities);

// Use the session to control Cortana
CortanaSession.FindElementByAccessibilityId("SearchTextBox").SendKeys("add");
```

# Avoiding splashscreens 
You can use a fixed delay. Below is the snippet of the workaround found in issue-ticket 213 that will work if the application under test enforce single instance and will simply bring up already launched instance when being re-launched.

(again, C# samples, but they illustrate the means)
```
DesiredCapabilities appCapabilities = new DesiredCapabilities();
appCapabilities.SetCapability("app", YourAppId);
WindowsDriver<WindowsElement> session = new WindowsDriver<WindowsElement>(new Uri(WindowsApplicationDriverUrl), appCapabilities);
Assert.IsNotNull(session);
// Wait for 5 seconds or however long it is needed for the right window to appear/for the splash screen to be dismissed
Thread.Sleep(TimeSpan.FromSeconds(5));
// When the application uses pre-launched existing instance, re-launching the application simply update 
// the current application window to whatever current main window belonging to the same application 
// process id
session.LaunchApp();
```

You can use SwitchTo() API to switch to the right main window once the correct main window is displayed. Below is a sample on how to switch window if your session is pointing to a wrong window such as the splash screen:

```
DesiredCapabilities appCapabilities = new DesiredCapabilities();
appCapabilities.SetCapability("app", YourAppId);
WindowsDriver<WindowsElement> session = new WindowsDriver<WindowsElement>(new Uri(WindowsApplicationDriverUrl), appCapabilities);
Assert.IsNotNull(session);
// Identify the current window handle. You can check through inspect.exe which window this is.
var currentWindowHandle = session.CurrentWindowHandle;
// Wait for 5 seconds or however long it is needed for the right window to appear/for the splash screen to be dismissed
Thread.Sleep(TimeSpan.FromSeconds(5));
// Return all window handles associated with this process/application.
// At this point hopefully you have one to pick from. Otherwise you can
// simply iterate through them to identify the one you want.
var allWindowHandles = session.WindowHandles;
// Assuming you only have only one window entry in allWindowHandles and it is in fact the correct one,
// switch the session to that window as follows. You can repeat this logic with any top window with the same
// process id (any entry of allWindowHandles)
session.SwitchTo().Window(allWindowHandles[0]);
```

# How to handle dynamically generated content (e.g. interacting with the Scrollbar)

The Alarm Clock Test serves as a good reference for this, and demonstrates two possible scenarios for handling elements in a list.

Scenario 1 - element pre-generated
As per the spec, we implicitly scroll elements within the view when they are selected--however this relies on the UI element being generated and accessible from the start. The Alarm Scenario test serves as a good reference for this. From the code snippet here:

minuteSelector.FindElementByName("55").Click();

The AlarmClock test automatically scrolls to the '55' element and selects it - as can be seen when the alarm GUI is set to 3:55 when the test is running.

Scenario 2 - element dynamically generated There exists the possibility of cases where the element UI is not generated until certain conditions are met (e.g. a list has to scroll to it's location first before element is generated). This will require automating in the steps to fulfill those requirements first before the element can be interacted with.

The Stopwatch Scenario serves as a good example for handling dynamically generated elements. From the code snippet here:

//the test scrolls to the location of the element before taking action on the desired element.
touchScreen.Scroll(lapListView.Coordinates, 0, -50); 

