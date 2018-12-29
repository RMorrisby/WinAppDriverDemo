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
