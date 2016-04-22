# Dynamic Web Server for Android #

The project create a web server on android based on starcore and SRPSHtmlEnvEngine, which are developed by srplab. It supports lua as dynamic script language embedded in web page. The lua script can access android java classes easily, which enables the web site provide much more functions. To use the code freely, an registered copy of starcore is needed. The lua script is simple and easy to use. When embedding lua in web page, uses <?lua> ... <?> tag.

# Create Project #

Create a web server project, add libraries of starcore to the project. As follow,

http://dynamic-web-server-for-android.googlecode.com/svn/wiki/images/workspace.PNG

# Init Starcore #

Using the following code to init starcore and import SRPSHtmlEnvEngine service.
```
        StarCoreFactoryPath.StarCoreCoreLibraryPath = "/data/data/"+getPackageName()+"/lib";
        StarCoreFactoryPath.StarCoreOperationPath= "/data/data/"+getPackageName()+"/files";
        StarCoreFactoryPath.StarCoreShareLibraryPath = "/data/data/"+getPackageName()+"/lib";
        
        String servicestr = null;
        try{
			InputStream dataSource = getAssets().open("SRPSHtmlEnvEngine.xml");
			int size=dataSource.available();
			byte[] buffer=new byte[size]; 
			dataSource.read(buffer); 
			dataSource.close();        
			servicestr=new String(buffer);        	
        }
        catch(Exception e){
        	
        }		
        starcore= StarCoreFactory.GetFactory();
		SrvGroup =starcore._InitSimpleEx(0,0);
		SrvGroup._ImportServiceFromXmlBuf(servicestr, true);
		SrvGroup._CreateService( ""," test", "123",5,0,0,0,0,0,"" );  
        Service = SrvGroup._GetService("root","123");
        Service._CheckPassword(false); 
```

## Create Web Server and Mount Root Path ##

```
        WebServerObject = Service._GetObject("SHtmlEnvSiteClass")._New()._Assign(new StarObjectClass(){
        	public void ScriptInit(StarObjectClass self,String ScriptName){
        		SrvGroup._Print("Init Script..."+ScriptName);
        		if( ScriptName.equals("lua") ){
        			SrvGroup._InitRaw("lua", Service);
        			StarObjectClass lua = Service._ImportRawContext("lua", "", false, "");
        			lua._Set("Activity", this);        			
        		}
        	}        	
        });  
        WebServerObject._Call("Mount","/","/sdcard/dynamicwebserver.srplab");
        WebServerObject._Set("RootUrl","/index.html");
        WebServerObject._Active();      
```

The ScriptInit callback function will be called when the first lua script is executed.
In this function, we set lua global variable "Activity" for lua script.

# Sample Web Page #

copy web page to "/sdcard/dynamicwebserver.srplab" directory using adb push method.

## first web page ##

The first web page is very simple. It only shows a string in the browser.

```
<!DOCTYPE html>
<html>
<body>
<h1><?lua echo([[hello from lua]]) /> </h1>
</body>
</html>
```

## second web page ##

The second page uses lua script to get machine info and returns it to web browser

```
<!DOCTYPE html>
<html>
<body>
<h1><?lua> 
--lua script
if luaisinit == nil then
   luaisinit = true
   SrvGroup = libstarcore._GetSrvGroup(0,0);
   Service = SrvGroup:_GetService("","")
   print(Service)  --print info will be shown in log info
   android = SrvGroup:_InitRaw("lua",Service)
   print(android)
   print("---------init from lua finish-----------")
end	
Build = Service:_ImportRawContext("java","android.os.Build",true,"")
print(Build)  --print info will be shown in log info
echo( Build.DEVICE)  --output to web page	
<?></h1>
</body>
</html>
```

# note #

Details about the SRPSHtmlEnvEngine, please refer to "http://www.srplab.com/Files/ewserver/ewserver.php?link=ewserver_functions.html"

# Screenshot #

![http://dynamic-web-server-for-android.googlecode.com/svn/wiki/images/screenshot.png](http://dynamic-web-server-for-android.googlecode.com/svn/wiki/images/screenshot.png)