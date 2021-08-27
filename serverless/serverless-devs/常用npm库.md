## UX交互相关
#### boxen
![boxen](https://img.alicdn.com/imgextra/i3/O1CN01kK2eRx1bbGCRZ4onK_!!6000000003483-2-tps-2785-785.png)
在终端中创建盒子，`一般用于结果类的展示`

#### emoji
[node-emoji](https://www.npmjs.com/package/node-emoji)
支持简单的表情符号。
```
var emoji = require('node-emoji')
emoji.get('coffee') // returns the emoji code for coffee (displays emoji on terminals that support it)
emoji.which(emoji.get('coffee')) // returns the string "coffee"
emoji.get(':fast_forward:')
```
![](https://camo.githubusercontent.com/4302c28b67f47195f638455c09ed32fa1683f0d6ef553c511a6789cc6cffcad1/68747470733a2f2f692e696d6775722e636f6d2f79496f355575782e706e67)


#### consola
优雅的Node.js和浏览器日志记录库
```
const consola = require('consola')
consola.success('Built!')
consola.info('Reporter: Some info')
consola.error(new Error('Foo'))
```
![](https://img.alicdn.com/imgextra/i3/O1CN01QUGjC21moxp7cXgiJ_!!6000000005002-2-tps-667-256.png)
- wrapConsole/wrapStd/wrapAll 将console,std等转换
```
function foo () {
  console.info('foo')
  console.warn('foo warn')
}
consola.wrapConsole()
```
- reporters
默认情况下，`FancyReporter`为现代的终端注册的，如果在CI等受限环境中运行，则使用`BasicReporter`。

#### chalk
[chalk](https://github.com/chalk/chalk)正确处理终端字符串样式
![](https://img.alicdn.com/imgextra/i1/O1CN013UPJlV1dQO1j7M5af_!!6000000003730-2-tps-1014-476.png)

#### ora
spinner加载器[ora](https://github.com/sindresorhus/ora)
```
import ora from 'ora';
const spinner = ora('Loading unicorns').start();
setTimeout(() => {
	spinner.color = 'yellow';
	spinner.text = 'Loading rainbows';
}, 1000);
```
![](https://img.alicdn.com/imgextra/i2/O1CN019MxhQy29TeYd0H5Yr_!!6000000008069-1-tps-347-192.gif)

#### progress
[progress](https://github.com/visionmedia/node-progress)一个灵活的进度条
```
var ProgressBar = require('progress');

var bar = new ProgressBar(':bar', { total: 10 });
var timer = setInterval(function () {
  bar.tick();
  if (bar.complete) {
    console.log('\ncomplete\n');
    clearInterval(timer);
  }
}, 100);

```
![progress](https://img.alicdn.com/imgextra/i1/O1CN01NFFXNk1I5DnCm9maY_!!6000000000841-1-tps-362-149.gif)

#### progress-estimator
[progress-estimator](https://github.com/bvaughn/progress-estimator)记录进度条并估计完成承诺所需的时间

```
const createLogger = require('progress-estimator');
const chalk = require('chalk');
const path = require('path');

let _logger = null;
const logger = (task, message, estimate) => {
  if (!_logger) {
    _logger = createLogger({
      storagePath: path.join(__dirname, '.progress-estimator'),
    });
  }
  return _logger(task, chalk.blue(message), {
    estimate,
  });
};

const task1 = new Promise(resolve => {
  setTimeout(() => {
    resolve({ success: true });
  }, 1200);
});

const task2 = new Promise(resolve => {
  setTimeout(() => {
    resolve({ success: true });
  }, 4200);
});

async function run() {
  const startTime = Date.now();
  console.log();
  console.log(chalk.blue('Some Tasks'));
  console.log();
  await logger(task1, 'Task 1', 1500);
  await logger(task2, 'Task 2', 600);
  const endTime = Date.now();
  const time = ((endTime - startTime) / 1000).toFixed(2);
  console.log();
  console.log(chalk.green(`✨ Done in ${time}s`));
  console.log();
}

run();
```
![progress-estimator](https://img.alicdn.com/imgextra/i1/O1CN01DREFjI1k1NiLya2JT_!!6000000004623-1-tps-495-197.gif)

#### listr
命令行任务列表
```
const execa = require('execa');
const Listr = require('listr');

const tasks = new Listr([
	{
		title: 'Git',
		task: () => {
			return new Listr([
				{
					title: 'Checking git status',
					task: () => execa.stdout('git', ['status', '--porcelain']).then(result => {
						if (result !== '') {
							throw new Error('Unclean working tree. Commit or stash changes first.');
						}
					})
				},
				{
					title: 'Checking remote history',
					task: () => execa.stdout('git', ['rev-list', '--count', '--left-only', '@{u}...HEAD']).then(result => {
						if (result !== '0') {
							throw new Error('Remote history differ. Please pull changes.');
						}
					})
				}
			], {concurrent: true});
		}
	},
	{
		title: 'Install package dependencies with Yarn',
		task: (ctx, task) => execa('yarn')
			.catch(() => {
				ctx.yarn = false;

				task.skip('Yarn not available, install it via `npm install -g yarn`');
			})
	},
	{
		title: 'Install package dependencies with npm',
		enabled: ctx => ctx.yarn === false,
		task: () => execa('npm', ['install'])
	},
	{
		title: 'Run tests',
		task: () => execa('npm', ['test'])
	},
	{
		title: 'Publish package',
		task: () => execa('npm', ['publish'])
	}
]);

tasks.run().catch(err => {
	console.error(err);
});

```
![listr](https://img.alicdn.com/imgextra/i2/O1CN01XdqDAn1SDbCCJzeXp_!!6000000002213-1-tps-1177-709.gif)


## 环境相关
#### cross-env
我们在自定义配置环境变量的时候，由于在不同的环境下，配置方式也是不同的。例如在window和linux下配置环境变量。
假设在设置环境变量`NODE_ENV=production`的时候
- window下配置
```
# node中常用的到的环境变量是NODE_ENV，首先查看是否存在 
set NODE_ENV 
#如果不存在则添加环境变量 
set NODE_ENV=production 
#环境变量追加值 set 变量名=%变量名%;变量内容 
set path=%path%;C:\web;C:\Tools 
#某些时候需要删除环境变量 
set NODE_ENV=
```
- linux下配置
```
#node中常用的到的环境变量是NODE_ENV，首先查看是否存在
echo $NODE_ENV
#如果不存在则添加环境变量
export NODE_ENV=production
#环境变量追加值
export path=$path:/home/download:/usr/local/
#某些时候需要删除环境变量
unset NODE_ENV
#某些时候需要显示所有的环境变量
env
```
3. cross-env在npm script中使用
```
{
  "scripts": {
    "build": "cross-env NODE_ENV=production webpack --config build/webpack.config.js"
  }
}
```
#### dotenv
开发过程中，经常遇到类似数据库密码，第三方服务密钥等敏感信息，我们往往不会写在代码中，通常是以环境变量的形式进行传递
- 创建`.env`文件
```
HOST=localhost
PORT=3000
MONGOOSE_URL=mongodb://localhost:27017/test
```
- 引入 dotenv 并使用
```
equire('dotenv').config({ path: '.env' })
// 使用
console.log(process.env.HOST) // localhost
```


## 进程相关
#### execa
这个包对`child_process`方法的增强在于
- Promise 接口
- 支持windows进程,使用[cross-spawn](https://github.com/moxystudio/node-cross-spaw)
- 缓冲区由`200KB`升到`100MB`
- 当父进程终止时候，清理产生的进程

1. exec默认的options为
```
{
    encoding: 'utf8', // 设置第三个参数的回调方法的参数stdout和stderr的字符编码，默认是utf8。如果这个值是buffer，则stdout和stderr为Buffer的实例
    timeout: 0, // 设置子进程执行的超时时间，默认是0，表示没有超时时间。当子进程执行的时间超过timeout设置的时间，则会向子进程发送killSignal结束该进程
    maxBuffer: 1024 * 1024, // 设置第三个参数的回调方法的参数stdout和stderr的最大长度，单位是Byte，默认是200KB
    killSignal: 'SIGTERM', // 设置结束子进程时发送的信号，默认是SIGTERM
    cwd: null, // 子进程工作目录
    env: null, // 子进程的环境变量
    uid: // 设置执行子进程的用户id。
    gid: // 设置执行子进程的用户id。
    shell: //  用于执行命令的 shell。 请参阅 shell 的要求和默认的 Windows shell。 默认值: Unix 上是 '/bin/sh'，Windows 上是 process.env.ComSpec
}
```
返回的是一个`buffer`的对象，默认为`200K`,当返回的数据默认超过默认大小时，会产生`Error: maxBuffer exceeded`异常。除了调大`maxBuffer`的选项，还可以使用spawn 可以解决这个问题。

2. spawn方法
使用spawn方法时，子进程一开始执行就会通过流返回数据，因此spawn适合子进程返回大量数据的情形。
{
    cwd: 同exec()。
    env: 同exec()。
    argv0: 显式设置子进程中argv[0]的值，如果没有声明则为command的值。
    stdio: 设置子进程中的标准输入输出，可以通过这个选项重定向子进程的标准输入输出。
    detached: 让子进程独立于父进程执行。
    uid: 同exec()。
    gid: 同exec()。
    shell: 如果设置成true，则使用shell执行command，这时候command的内容就会被bash解析，意味着也能使用复杂的命令行。在UNIX上可以设置成/bin/sh，在Windows上可以设置成cmd.exe。这个shell必须能解析UNIX中的-c跳转和Windows中的/d /s /c。默认是false。
}

3. 有无创建子`shell`区别
- 没有创建子`shell`的效率更高
- 一些操作，比如`I/O`重定向，文件`glob`等不支持



#### bash和sh的区别
- SH
sh是UNIX标准的默认shell,属于系统管理的shell
- BASH
是`linux`标准的默认`shell`, `bash`是完全兼容`sh`，反过来`bash`脚本在`sh`上运行语法容易报错。

为什么会出现`sh`?
因为bash过于复杂，`/bin/sh`可以获取更快的脚本执行速度。
![bash](https://img.alicdn.com/imgextra/i1/O1CN01HLy6nt22HYmpvSVQA_!!6000000007095-0-tps-368-246.jpg)


## 其他工具
#### portfinder
[portfinder](https://github.com/http-party/node-portfinder) 会自动寻找`8000至65535`内的端口
- 基本使用
```
const portfinder = require('portfinder');
portfinder.getPortPromise().then(() => console.log)
```
- 端口范围配置
portfinder.basePort = 3000;    // default: 8000
portfinder.highestPort = 3333; // default: 65535

#### envinfo
生成故障排除软件问题(如操作系统、二进制版本、浏览器、已安装语言等)时所需的通用详细信息的报告
```
import envinfo from 'envinfo';

envinfo.run(
    {
        System: ['OS', 'CPU'],
        Binaries: ['Node', 'Yarn', 'npm'],
        Browsers: ['Chrome', 'Firefox', 'Safari'],
        npmPackages: ['styled-components', 'babel-plugin-styled-components'],
    },
    { json: true, showNotFound: true }
).then(env => console.log(env));
```
生成的报告如下
```
{
    "System": {
        "OS": "macOS High Sierra 10.13",
        "CPU": "x64 Intel(R) Core(TM) i7-4870HQ CPU @ 2.50GHz"
    },
    "Binaries": {
        "Node": {
            "version": "8.11.0",
            "path": "~/.nvm/versions/node/v8.11.0/bin/node"
        },
        "Yarn": {
            "version": "1.5.1",
            "path": "~/.yarn/bin/yarn"
        },
        "npm": {
            "version": "5.6.0",
            "path": "~/.nvm/versions/node/v8.11.0/bin/npm"
        }
    },
    "Browsers": {
        "Chrome": {
            "version": "67.0.3396.62"
        },
        "Firefox": {
            "version": "59.0.2"
        },
        "Safari": {
            "version": "11.0"
        }
    },
    "npmPackages": {
        "styled-components": {
            "wanted": "^3.2.1",
            "installed": "3.2.1"
        },
        "babel-plugin-styled-components": "Not Found"
    }
}
```
#### read-pkg-up
[read-pkg-up](https://github.com/sindresorhus/read-pkg-up)
读取最近的package.json文件
```
import {readPackageUpAsync} from 'read-pkg-up';
console.log(await readPackageUpAsync());
```
