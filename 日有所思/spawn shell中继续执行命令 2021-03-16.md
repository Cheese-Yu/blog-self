# spawn shell中继续执行命令
```javascript
function doSpawn(commands) {
  if (!commands || !commands.length) return;
  const shell = spawn("adb", ["shell", ""]);
  shell.stdout.on("data", (data) => {
    console.log(`stdout: ${data}`);
    if (commands.length > 0) {
      shell.stdin.write(commands[0] + "\n");
      commands.shift();
    }
  });

  shell.stderr.on("data", (data) => {
    console.log(`stderr: ${data}`);
  });

  shell.on("close", (code) => {
    console.log(`子进程退出码：${code}`);
  });
}
```
  
<br />
> 语雀地址 https://www.yuque.com/cheeseyu/fullstack/fkp9e9