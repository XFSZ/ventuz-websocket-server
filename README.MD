# 如何在VENTUZ下使用C#版的Socketio 
**使用方法**  
    一、新建脚本  
    二、在脚本的左下角References选项卡中 找到PAC Assemblies  点击添加一下三个 dll  dll必须得是4.5 
    ![alt 截图1](./1.png)   
    三、在脚本中需要引入 using Newtonsoft.Json.Linq; using SocketIOClient;  
    ![alt 截图1](./2.png)  
    四、VENTUZ脚本总是出一些奇奇怪怪的 编译错误 例如： await 多余问题 需要手动加上new System.EventHandler 且不支持lamda表达式  dll文件的各种不兼容  
    五、 更新renderer需要 changed = true;  
    ![alt 截图1](./3.png) 
    
```cs 
using System;
using System.Collections;
using System.Collections.Generic;
using System.Linq;
using Ventuz.Kernel;
using Newtonsoft.Json.Linq;
using SocketIOClient;
using System.Threading.Tasks;


public class Script : ScriptBase, System.IDisposable
{
    
    // This member is used by the Validate() method to indicate
    // whether the Generate() method should return true or false
    // during its next execution.
    private bool changed;
    private JToken result;
	public	SocketIO client= new SocketIO("http://127.0.0.1:3000/");
	public string latestMyName = "who";
    public string latestRoom = "X-Art";
	public		JToken res;
	public JToken res2;
   // public	System.Timers.Timer t = new System.Timers.Timer();
   // private	bool init = false;
	
    // This Method is called if the component is loaded/created.
    public Script()
    {
		//client = new SocketIO("http://127.0.0.1:3000/");
        // Note: Accessing input or output properties from this method
        // will have no effect as they have not been allocated yet.
    }
    
    // This Method is called if the component is unloaded/disposed
    public virtual void Dispose()
    {
    }
    
    // This Method is called if an input property has changed its value
  //  public override void Validate()
 //   {
        // Remember: set changed to true if any of the output 
        // properties has been changed, see Generate()
 //   }
    
    // This Method is called every time before a frame is rendered.
    // Return value: if true, Ventuz will notify all nodes bound to this
    //               script node that one of the script's outputs has a
    //               new value and they therefore need to validate. For
    //               performance reasons, only return true if output
    //               values really have been changed.
    public override bool Generate()
    {
		
			changed = true;
        if (changed)
        {
	//				if(init){
	//		t.Interval = 1000;
	//		t.Elapsed += Tick;
	//		t.Start();
	//		init = false;
	//				}
           // changed = false;
			//
            return true;
		}

	


        return false;
	}
	
	
	// This Method is called if an input property has changed its value
	public override void Validate()
	{   
	
		// Remember: set changed to true if any of the output 
		// properties has been changed, see Generate()
		if ((string.IsNullOrEmpty(this.MyName) == false && latestMyName.Equals(this.MyName) == false))
		{
			latestMyName = this.MyName;
			//OnReGetData(-1);
		}	
		if ((string.IsNullOrEmpty(this.Room) == false && latestRoom.Equals(this.Room) == false))
		{
			latestRoom = this.Room;
			//OnReGetData(-1);
		}
	//	else if (isNeedInitData == true)
	//	{
			//OnReGetData(-1);
	//	}
	}
	
	// This Method is called if the function/method Method1 is invoked by the user or a bound event.
	// Return true, if this component has to be revalidated!
	public bool OnConnectHost(int arg)
	{
	//	client = new SocketIO("http://127.0.0.1:3000/");
		client.On("users", response =>
			{
			string text = response.GetValue<string>();
		//	OutPutTest = text;
			});
		client.On("getCameraValue", response =>
			{
			res = response.GetValue();
		//	Alpha = Convert.ToDouble(res.Value<string>("alpha"))*(180/Math.PI) ;
			Alpha = Convert.ToDouble(res.Value<string>("alpha")) *(180/Math.PI)-90;
			Beta = Convert.ToDouble(res.Value<string>("beta"))*(180/Math.PI);
			Radius = Convert.ToDouble(res.Value<string>("radius"))*(0.01d);
			res2 = res.Value<JToken>("target");
			//VLog.Info(res2.Value<string>("x").ToString());

			TargetX  = Convert.ToDouble(res2.Value<string>("x"));
			TargetY  = Convert.ToDouble(res2.Value<string>("y"));
			TargetZ  = Convert.ToDouble(res2.Value<string>("z"));
			
			res2 = res.Value<JToken>("position");
			PositionX =  Convert.ToDouble(res2.Value<string>("x"));
			PositionY =  Convert.ToDouble(res2.Value<string>("y"));
			PositionZ =  Convert.ToDouble(res2.Value<string>("z"));
			
		//	changed = true;
			});
     	
		client.OnPing += Socket_OnPing;
		client.OnPong += Socket_OnPong;
		client.OnConnected += new System.EventHandler(Backval);
		client.ConnectAsync();

		return true;
	}
	public async void Backval(object sender, EventArgs e) {
	
		string jsonstr = @"{'name':'"+latestMyName+"','room':'"+latestRoom+"'}";
	//	OutPutTest = jsonstr;
		await client.EmitAsync("users", new
			{
					home="lates"
			}, "AYU");
		await client.EmitAsync("joinroom", response =>
			{
	
			}, jsonstr);

	}

	private  void Socket_OnPing(object sender, EventArgs e)
	{
		Console.WriteLine("Ping");
	}

	private  void Socket_OnPong(object sender, TimeSpan e)
	{
		Console.WriteLine("Pong: " + e.TotalMilliseconds);
		Latency = e.TotalMilliseconds.ToString();
	}
	public void Tick(object sender, EventArgs e) {
		client.EmitAsync("Ping");
	}
	// This Method is called if the function/method DisconnectHost is invoked by the user or a bound event.
	// Return true, if this component has to be revalidated!
	public bool OnDisconnectHost()
	{
		client.EmitAsync("disconnect", response =>
			{
		//	string text = response.GetValue<string>();
		VLog.Info("disconnect");
			});
		return true;
	}


}

```

**依赖**：  
https://github.com/doghappy/socket.io-client-csharp

**excel 转 json**
```cs 
	 void ParseFile(string excelPath)
	{
		VLog.Info(ExcelURL);
		excelPath = ExcelURL;
		Stream stream = new FileStream(excelPath, FileMode.Open, FileAccess.Read);
		var index = excelPath.LastIndexOf('.');
		var extensionName = excelPath.Substring(index + 1);

		VLog.Info("2");

		var reader = ExcelReaderFactory.CreateOpenXmlReader(stream);
		if(reader == null)
		{
			VLog.Info("文件错误");
			return;
		}

		var dt = reader.AsDataSet();
		// 当 将第一行作为表头是使用
/*		var dt = reader.AsDataSet(new ExcelDataSetConfiguration() {
				ConfigureDataTable = (_) => new ExcelDataTableConfiguration() {
					UseHeaderRow = true
					}
				});
				*/    

		dt.AcceptChanges();
		VLog.Info(dt.ToString());
		string json = JsonConvert.SerializeObject(dt, Formatting.Indented);
		VLog.Info(json.ToString());
		JsonStr = json;
	//	if(dt.Tables.Count < 2)
	//	{
//			Console.WriteLine("文件：" + excelPath + " 找不到第二张表");
//			return;
//		}

	//	var table = dt.Tables[1];
	//	var cols = table.Columns.Count;

	//	if(cols <= 1)
	//	{
//			Console.WriteLine("文件：" + excelPath + " 需要至少有两列");
//			return;
//		}

	}


```


## 服务器server 总入口
```cs
 private bool changed;
    public WebSocketServer wssv = new WebSocketServer (4649);
    public	int tempPort = 4649;
	public	string tempSendMessage = "";

    // This Method is called if the component is loaded/created.
    public Script()
    {
			
        // Note: Accessing input or output properties from this method
        // will have no effect as they have not been allocated yet.
    }
    
    // This Method is called if the component is unloaded/disposed
    public virtual void Dispose()
    {
		VLog.Info("-");
		wssv.Stop();
		
    }
    
    // This Method is called if an input property has changed its value
    public override void Validate()
    {
		
		if ( tempPort!=this.Port)
		{
			 wssv.Stop();
			tempPort = this.Port;
			startSocketServer();
		}	
			if (string.IsNullOrEmpty(this.SendMessageStr) == false && tempSendMessage.Equals(this.SendMessageStr) == false)
			{
			//	wssv.WebSocketServices["/Chat"].Sessions.Broadcast("what");
			wssv.WebSocketServices.Broadcast(tempSendMessage);
			//ReceiveMessageStr = Chat.ReceiveMessageStr;
			}
		

        // Remember: set changed to true if any of the output 
        // properties has been changed, see Generate()
    }
    
    // This Method is called every time before a frame is rendered.
    // Return value: if true, Ventuz will notify all nodes bound to this
    //               script node that one of the script's outputs has a
    //               new value and they therefore need to validate. For
    //               performance reasons, only return true if output
    //               values really have been changed.
    public override bool Generate()
    {

	ReceiveMessageStr = Chat.ReceiveMessageStr;
	//	VLog.Info("0");
	
     //   if (changed)
     //   {
      //      changed = false;
      //      return true;
      //  }

       // return false;
		return true;
	}
	
	// This Method is called if the function/method Initserver is invoked by the user or a bound event.
	// Return true, if this component has to be revalidated!
	public bool OnInitServer(int arg)
	{
		startSocketServer();
		return true;
	}
	
	// This Method is called if the function/method StopServer is invoked by the user or a bound event.
	// Return true, if this component has to be revalidated!
	public bool OnStopServer(int arg)
	{   wssv.Stop();
		return true;
	}
    
	public void startSocketServer(){
		wssv = new WebSocketServer ( tempPort);
		//	wssv.AddWebSocketService<Echo>("/Echo");
		wssv.AddWebSocketService<Chat>("/Chat");
		wssv.Start();
		if (wssv.IsListening) {
			VLog.Info("Listening on port {0}, and providing WebSocket services:" + tempPort);
			foreach (var path in wssv.WebSocketServices.Paths){
				VLog.Info("- {0}" + path);
			}
		}
	}
}
public class Echo : WebSocketBehavior
{
	
	public	string SendMessageStr {get;set;}
	public	string ReceiveMessageStr {get;set;}
	public Echo(){
	IgnoreExtensions = true;
	}
	public Echo(string sendMessageStr){
		this.SendMessageStr = sendMessageStr;
		VLog.Info("SendMessageStr : "+SendMessageStr);
		Send(SendMessageStr);
		VLog.Info("ReceiveMessageStr : "+ReceiveMessageStr);
	}
	protected override void OnMessage (MessageEventArgs e)
	{
		VLog.Info(e.Data.ToString());
		//	var name = Context.QueryString["name"];
		//	Send(!name.IsNullOrEmpty() ? String.Format("\"{0}\" to {1}", e.Data, name) : e.Data);
		Send(SendMessageStr);
		ReceiveMessageStr = e.Data;
	}
}

public  class Chat : WebSocketBehavior
{

	public static	string ReceiveMessageStr = "";
	public Chat(){	IgnoreExtensions = true;}

	protected override void OnClose (CloseEventArgs e)
	{
		Sessions.Broadcast("close");
	}

	protected override void OnMessage (MessageEventArgs e)
	{
		VLog.Info(e.Data.ToString());
		VLog.Info("OK");
		ReceiveMessageStr = e.Data.ToString();
		
	}
	public  void Broadcast(string msg)
	{

			Sessions.Broadcast(msg);
	}
//	protected override void OnOpen ()
//	{
//	Sessions.Broadcast("close");
//	}
```


## 子项 终端
```cs
 private bool changed;
	string tempMessageStr = "";
	string tempButtonName = "";
    // This Method is called if the component is loaded/created.
    public Script()
    {
        // Note: Accessing input or output properties from this method
        // will have no effect as they have not been allocated yet.
    }
    
    // This Method is called if the component is unloaded/disposed
    public virtual void Dispose()
    {
    }
    
    // This Method is called if an input property has changed its value
    public override void Validate()
    {
		if (string.IsNullOrEmpty(this.MessageStr) == false && tempMessageStr.Equals(this.MessageStr) == false)
		{
			tempMessageStr = this.MessageStr;
			parseMessageStr(tempMessageStr);
		}
		if (string.IsNullOrEmpty(this.ButtonName) == false && tempButtonName.Equals(this.ButtonName) == false)
		{
			tempButtonName = this.ButtonName;
		//	BottonEvent();
		}
        // Remember: set changed to true if any of the output 
        // properties has been changed, see Generate()
    }
    
    // This Method is called every time before a frame is rendered.
    // Return value: if true, Ventuz will notify all nodes bound to this
    //               script node that one of the script's outputs has a
    //               new value and they therefore need to validate. For
    //               performance reasons, only return true if output
    //               values really have been changed.
    public override bool Generate()
    {
     //   if (changed)
     //   {
      //      changed = false;
            return true;
      //  }

      //  return false;
    }
	
	public void parseMessageStr(string messageStr){
		JObject jo = (JObject) JsonConvert.DeserializeObject(messageStr);
		string buttonName = jo.Value<string>("buttonName").ToString();
		//	VLog.Info("buttonName", jo.Value<string>("buttonName").ToString());
		//	VLog.Info("transData", jo.Value<JObject>("transData").ToString());
		JObject res = jo.Value<JObject>("transData");
		string btnEvent = res.Value<string>(buttonName);
		VLog.Info("btnEvent", btnEvent);
		if(btnEvent.Equals("True") && buttonName.Equals(tempButtonName)){
			BottonEvent();
		}

		   
	}
```