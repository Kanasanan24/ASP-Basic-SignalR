# เริ่มต้นการใข้งาน ASP.NET Core ด้วย SignalR
1. สร้าง ASP.NET ด้วย ASP.NET Core Web App (Razor Page) ใช้เป็น .NET 8
2. Right Click ที่ Solution Explorer ตรง project แล้วเลือก Add > Client-Side Library
3. ใช้ provider เป็น unpkg แล้วพิมพ์ @microsoft/signalr@latest
4. เลือก radio box เป็นแบบ Choose specific file เราหา Files > dist > browser
  check แค่ signalr.js และ signalr.min.js
5. Set Target Location เป็น wwwroot/js/signalr/
6. Install


## สร้าง SignalR hub

hub เป็น class ทำหน้าที่เป็น high-level pipeline ในการรับมือการสื่อสารระหว่าง client-server
1. ให้เราสร้าง Hubs Folder ที่ root ของ project
2. สร้าง ChatHub class แล้วสืบทอด Hub class (เป็น class จัดการ connection, groups และการ message) จากนั้นสร้าง method ดังนี้
```
  public async Task SendMessage(string user, string message)
  {
    await Clients.AllSendAsync("ReceiveMessage", user, message);
  }
```
- SendMessage เป็น method ที่เชื่อมกับ client เพื่อที่จะส่ง message ให้กับทุกๆ client ซึ่ง SignalR จะต้องเป็นแบบ async เพื่อให้มี
ความสามารถในการปรับขนาดที่สูง

## การ Configure SignalR

ตัว SignalR server ต้อง configure เพื่อที่จะส่ง SignalR request ไปยัง SignalR server ได้
ซึ่งเราจะต้องเพิ่ม Service และ Middleware ของ SignalR ใน Program.cs

```
using SignalRChat.Hubs;

builder.Services.AddSignalR();

app.MapHub<ChatHub>("/charHub"); // เพิ่มบน app.Run();
```

## เพิ่ม Code ทางฝั่ง Client

ใน Pages/Index.cshtml เพิ่ม code นี้

```
@page
<div class="container">
  <div class="row p-1">
    <div class="col-1">User</div>
    <div class="col-5"><input type="text" id="userInput" /></div>
  </div>
  <div class="row p-1">
    <div class="col-1">Message</div>
    <div class="col-5"><input type="text" class="w-100" id="messageInput" /></div>
  </div>
  <div class="row p-1">
    <div class="col-6 text-end">
      <input type="button" id="sendButton" value="Send Message" />
    </div>
  </div>
  <div class="row p-1">
    <div class="col-6">
      <hr />
    </div>
  </div>
  <div class="row p-1">
    <div class="col-6">
      <ul id="messagesList"></ul>
    </div>
  </div>
</div>
<script src="~/js/signalr/dist/browser/signalr.js"></script>
<script src="~/js/chat.js"></script>
```
- หลักๆ คือจะเป็นการสร้าง web chat ที่ทุกๆ client จะเห็นกันหมดโดย message ที่ client พิมพ์จะไปอยู่ในส่วน
ของ <ul id="messagesList"></ul> รวมถึง import library ของ signalr เข้ามาทำงานด้วย

## สร้าง File chat.js ใน wwwroot/js

เมื่อสร้างไฟล์แล้วให้วาง code ดังนี้

```
"use strict";

var connection = new signalR.HubConnectionBuilder().withUrl("/chatHub").build();

//Disable the send button until connection is established.
document.getElementById("sendButton").disabled = true;

connection.on("ReceiveMessage", function (user, message) {
    var li = document.createElement("li");
    document.getElementById("messagesList").appendChild(li);
    // We can assign user-supplied strings to an element's textContent because it
    // is not interpreted as markup. If you're assigning in any other way, you 
    // should be aware of possible script injection concerns.
    li.textContent = `${user} says ${message}`;
});

connection.start().then(function () {
    document.getElementById("sendButton").disabled = false;
}).catch(function (err) {
    return console.error(err.toString());
});

document.getElementById("sendButton").addEventListener("click", function (event) {
    var user = document.getElementById("userInput").value;
    var message = document.getElementById("messageInput").value;
    connection.invoke("SendMessage", user, message).catch(function (err) {
        return console.error(err.toString());
    });
    event.preventDefault();
});
```

- เป็นการสร้างแล้วเริ่มต้น connection
- มีการรับมือกับ submit button ให้ส่ง message ไปที่ hub
- เพิ่ม connection object เพื่อรับ message จาก hub แล้วเพิ่มใน list ul ตามข้างต้น

### สามารถ Run ได้เลย