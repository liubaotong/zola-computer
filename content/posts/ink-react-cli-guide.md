+++
title = "Ink 完全指南 - 用 React 构建漂亮的命令行应用"
date = "2026-04-03T15:00:00+08:00"
updated = "2026-04-03T15:00:00+08:00"

[taxonomies]
tags = ["React", "CLI", "Ink", "Node.js", "TypeScript", "前端工具"]
categories = ["前端开发", "工具链"]

[extra]
summary = "全面介绍 Ink - 一个用 React 构建命令行界面的库。从基础概念到实战开发，涵盖核心组件、常用生态组件、交互式 UI 开发、最佳实践和完整项目示例。"
author = "博主"
+++

# Ink 完全指南 - 用 React 构建漂亮的命令行应用

## 什么是 Ink？

Ink 是一个 React 渲染器，让你能够使用 React 组件来构建和测试命令行界面（CLI）输出。它提供了与浏览器中 React 相同的基于组件的 UI 构建体验，但是针对终端环境进行了优化。

用官方文档的话说：**"React for CLIs. Build and test your CLI output using components."**

### 核心特性

- **完整的 React 支持**：所有 React 功能都可以使用（hooks、state、context、suspense 等）
- **Flexbox 布局**：使用类似 CSS 的布局方式，基于 Facebook 的 Yoga 布局引擎
- **键盘输入处理**：通过 `useInput` hook 处理用户交互
- **组件化架构**：可复用的组件设计，易于维护和测试
- **TypeScript 支持**：开箱即用的类型定义
- **丰富的生态**：大量第三方 Ink 组件可用

### 谁在使用 Ink？

Ink 已经被许多知名项目采用：

- **Claude Code** - Anthropic 的 AI 编程助手 CLI
- **Gemini CLI** - Google 的 AI 编程工具
- **GitHub Copilot CLI** - GitHub 的 AI 编程助手
- **Cloudflare Wrangler** - Cloudflare Workers CLI
- **Prisma** - 统一数据层工具
- **Gatsby** - 现代 Web 框架
- **Tap** - JavaScript 测试框架

## 安装与快速开始

### 基础安装

```bash
# 创建项目
mkdir my-cli && cd my-cli
npm init -y

# 安装核心依赖
npm install ink react
npm install -D typescript @types/react
```

### TypeScript 配置

创建 `tsconfig.json`：

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "lib": ["ES2020", "DOM"],
    "jsx": "react-jsx",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "dist",
    "rootDir": "src"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### package.json 配置

```json
{
  "name": "my-cli",
  "version": "1.0.0",
  "type": "module",
  "bin": "dist/cli.js",
  "scripts": {
    "build": "tsc",
    "dev": "tsc --watch",
    "start": "node dist/cli.js"
  }
}
```

**注意**：`"type": "module"` 很重要，因为 Ink 使用 ESM 模块。

### 第一个 Ink 应用

创建入口文件 `src/cli.ts`：

```typescript
#!/usr/bin/env node
import React from 'react';
import {render} from 'ink';
import App from './app.js';

render(React.createElement(App));
```

创建主组件 `src/app.tsx`：

```typescript
import React from 'react';
import {Box, Text} from 'ink';

const App: React.FC = () => {
	return (
		<Box flexDirection="column" padding={1}>
			<Text bold color="green">Hello, Ink!</Text>
			<Text>这是一个用 React 构建的 CLI 应用</Text>
		</Box>
	);
};

export default App;
```

构建并运行：

```bash
npm run build
npm start
```

你会在终端中看到绿色的 "Hello, Ink!" 文字！

## 核心概念

### 1. Box 组件 - 布局基石

`Box` 是 Ink 中最基础的布局组件，基于 Flexbox 模型。

```typescript
import {Box, Text} from 'ink';

// 水平布局
<Box>
	<Text>左边</Text>
	<Text>右边</Text>
</Box>

// 垂直布局
<Box flexDirection="column">
	<Text>第一行</Text>
	<Text>第二行</Text>
</Box>

// 固定宽度
<Box width={20}>
	<Text>固定 20 字符宽度</Text>
</Box>

// 百分比宽度
<Box width="50%">
	<Text>占 50% 宽度</Text>
</Box>

// 内边距
<Box paddingX={2} paddingY={1}>
	<Text>有内边距的盒子</Text>
</Box>

// 对齐方式
<Box justifyContent="center" alignItems="center">
	<Text>居中的文本</Text>
</Box>
```

### 2. Text 组件 - 文字样式

`Text` 组件用于显示和样式化文字：

```typescript
import {Text} from 'ink';

// 颜色
<Text color="red">红色文字</Text>
<Text color="green">绿色文字</Text>
<Text color="blue">蓝色文字</Text>

// 支持的颜色值：
// black, red, green, yellow, blue, magenta, cyan, white, gray

// 粗体和斜体
<Text bold>粗体文字</Text>
<Text italic>斜体文字</Text>
<Text underline>下划线</Text>
<Text strikethrough>删除线</Text>

// 透明度
<Text dimColor>半透明文字</Text>

// 背景色
<Text backgroundColor="red">红色背景</Text>

// 组合使用
<Text bold color="green" backgroundColor="black">
	绿色粗体，黑色背景
</Text>
```

### 3. 边框样式

Box 支持多种边框样式：

```typescript
import {Box, Text} from 'ink';

// 单线边框
<Box borderStyle="single" paddingX={2}>
	<Text>单线边框</Text>
</Box>

// 双线边框
<Box borderStyle="double" paddingX={2}>
	<Text>双线边框</Text>
</Box>

// 圆角边框
<Box borderStyle="round" paddingX={2}>
	<Text>圆角边框</Text>
</Box>

// 粗边框
<Box borderStyle="bold" paddingX={2}>
	<Text>粗边框</Text>
</Box>

// 其他样式：
// single-double, double-single, classic
```

## 核心 Hooks

### 1. useInput - 键盘输入处理

`useInput` 是处理用户键盘输入的核心 hook：

```typescript
import {useInput} from 'ink';

const App = () => {
	useInput((input, key) => {
		// input 是输入的字符
		// key 是按键信息对象
		
		if (input === 'q') {
			console.log('按下了 q');
		}
		
		if (key.escape) {
			console.log('按下了 Escape');
		}
		
		if (key.upArrow) {
			console.log('按下了上箭头');
		}
		
		if (key.downArrow) {
			console.log('按下了下箭头');
		}
		
		if (key.ctrl && input === 'c') {
			console.log('按下了 Ctrl+C');
		}
		
		if (key.return) {
			console.log('按下了 Enter');
		}
		
		if (key.backspace) {
			console.log('按下了 Backspace');
		}
	});

	return <Text>按 q 退出</Text>;
};
```

**实际应用 - 菜单导航**：

```typescript
import React, {useState} from 'react';
import {Box, Text, useInput, useApp} from 'ink';

const items = ['首页', '设置', '关于', '退出'];

const Menu = () => {
	const {exit} = useApp();
	const [selectedIndex, setSelectedIndex] = useState(0);

	useInput((input, key) => {
		if (input === 'q') {
			exit();
		}

		if (key.upArrow) {
			setSelectedIndex(prev => Math.max(0, prev - 1));
		}

		if (key.downArrow) {
			setSelectedIndex(prev => Math.min(items.length - 1, prev + 1));
		}

		if (key.return) {
			console.log(`选择了：${items[selectedIndex]}`);
		}
	});

	return (
		<Box flexDirection="column">
			{items.map((item, index) => (
				<Text key={index} color={index === selectedIndex ? 'green' : 'white'}>
					{index === selectedIndex ? '▶ ' : '  '}
					{item}
				</Text>
			))}
		</Box>
	);
};
```

### 2. useApp - 应用控制

`useApp` 提供对 Ink 应用实例的访问：

```typescript
import {useApp} from 'ink';

const App = () => {
	const {exit} = useApp();

	const handleExit = () => {
		// 退出应用
		exit();
		
		// 或者传递值
		exit(new Error('出错了'));
		exit({code: 0, message: '成功'});
	};

	return <Text>按 q 退出</Text>;
};
```

### 3. useStdout / useStdin / useStderr - 流访问

```typescript
import {useStdout, useStdin, useStderr} from 'ink';

const App = () => {
	const {stdout, write} = useStdout();
	const {stdin} = useStdin();
	const {stderr, write: writeError} = useStderr();

	// 直接写入 stdout
	write('直接输出到 stdout\n');

	// 写入 stderr
	writeError('错误信息\n');

	return <Text>流访问示例</Text>;
};
```

### 4. useState + useEffect - React 状态管理

完全支持 React hooks：

```typescript
import React, {useState, useEffect} from 'react';
import {Text} from 'ink';

// 计数器
const Counter = () => {
	const [count, setCount] = useState(0);

	useEffect(() => {
		const timer = setInterval(() => {
			setCount(prev => prev + 1);
		}, 1000);

		return () => clearInterval(timer);
	}, []);

	return <Text>{count} 秒已过</Text>;
};

// 数据获取
const DataFetcher = () => {
	const [data, setData] = useState(null);
	const [loading, setLoading] = useState(true);

	useEffect(() => {
		fetch('https://api.example.com/data')
			.then(res => res.json())
			.then(data => {
				setData(data);
				setLoading(false);
			});
	}, []);

	if (loading) {
		return <Text>加载中...</Text>;
	}

	return <Text>{JSON.stringify(data)}</Text>;
};
```

### 5. Static 组件 - 永久输出

`Static` 组件用于保留已渲染的输出，不被后续更新覆盖：

```typescript
import React, {useState, useEffect} from 'react';
import {Box, Text, Static} from 'ink';

const BuildLog = () => {
	const [logs, setLogs] = useState<string[]>([]);
	const [current, setCurrent] = useState('正在构建...');

	useEffect(() => {
		const steps = [
			'✓ 编译 TypeScript',
			'✓ 打包资源',
			'✓ 运行测试',
			'✓ 部署完成'
		];

		steps.forEach((step, index) => {
			setTimeout(() => {
				setLogs(prev => [...prev, step]);
				if (index === steps.length - 1) {
					setCurrent('构建完成！');
				}
			}, index * 1000);
		});
	}, []);

	return (
		<Box flexDirection="column">
			<Static items={logs}>
				{(log) => <Text color="green">{log}</Text>}
			</Static>
			<Text color="cyan">{current}</Text>
		</Box>
	);
};
```

## 常用 Ink 组件生态

### 1. ink-text-input - 文本输入

```bash
npm install ink-text-input
```

```typescript
import React, {useState} from 'react';
import {Box, Text} from 'ink';
import TextInput from 'ink-text-input';

const InputDemo = () => {
	const [value, setValue] = useState('');
	const [submitted, setSubmitted] = useState<string[]>([]);

	const handleSubmit = (value: string) => {
		if (value.trim()) {
			setSubmitted(prev => [...prev, value]);
			setValue('');
		}
	};

	return (
		<Box flexDirection="column">
			<Box>
				<Text color="cyan">$ </Text>
				<TextInput
					value={value}
					onChange={setValue}
					onSubmit={handleSubmit}
					placeholder="输入命令..."
				/>
			</Box>

			{submitted.length > 0 && (
				<Box flexDirection="column" marginTop={1}>
					<Text bold>历史记录：</Text>
					{submitted.map((item, index) => (
						<Text key={index}>• {item}</Text>
					))}
				</Box>
			)}
		</Box>
	);
};
```

### 2. ink-select-input - 选择菜单

```bash
npm install ink-select-input
```

```typescript
import React, {useState} from 'react';
import {Box, Text} from 'ink';
import SelectInput from 'ink-select-input';

const SelectDemo = () => {
	const [selected, setSelected] = useState<string | null>(null);

	const items = [
		{label: '🚀 React', value: 'react'},
		{label: '💚 Vue', value: 'vue'},
		{label: '💙 Angular', value: 'angular'},
		{label: '🧡 Svelte', value: 'svelte'},
	];

	const handleSelect = (item: any) => {
		setSelected(item.value);
	};

	return (
		<Box flexDirection="column">
			<SelectInput
				items={items}
				onSelect={handleSelect}
			/>

			{selected && (
				<Text color="green">✓ 已选择：{selected}</Text>
			)}
		</Box>
	);
};
```

### 3. ink-spinner - 加载动画

```bash
npm install ink-spinner
```

```typescript
import React from 'react';
import {Box, Text} from 'ink';
import Spinner from 'ink-spinner';

const LoadingDemo = () => {
	return (
		<Box flexDirection="column">
			<Box>
				<Text color="cyan">
					<Spinner type="dots" />
				</Text>
				<Text> 正在加载...</Text>
			</Box>

			<Box>
				<Text color="green">
					<Spinner type="line" />
				</Text>
				<Text> 处理中...</Text>
			</Box>
		</Box>
	);
};
```

### 4. ink-stepper - 分步向导

```bash
npm install ink-stepper
```

```typescript
import React from 'react';
import {Box, Text} from 'ink';
import TextInput from 'ink-text-input';
import {Stepper, Step} from 'ink-stepper';

const WizardDemo = () => {
	const [formData, setFormData] = useState({
		name: '',
		email: '',
	});

	return (
		<Stepper
			onComplete={() => console.log('完成！')}
			showProgress={true}
		>
			<Step name="姓名">
				{(ctx) => (
					<Box flexDirection="column">
						<Text bold>步骤 1：输入姓名</Text>
						<TextInput
							value={formData.name}
							onChange={(val) => setFormData({...formData, name: val})}
							onSubmit={() => ctx.goNext()}
						/>
					</Box>
				)}
			</Step>

			<Step name="邮箱">
				{(ctx) => (
					<Box flexDirection="column">
						<Text bold>步骤 2：输入邮箱</Text>
						<TextInput
							value={formData.email}
							onChange={(val) => setFormData({...formData, email: val})}
							onSubmit={() => {
								if (formData.email.includes('@')) {
									ctx.goNext();
								}
							}}
						/>
					</Box>
				)}
			</Step>

			<Step name="确认">
				{(ctx) => (
					<Box flexDirection="column">
						<Text bold>确认信息：</Text>
						<Text>姓名：{formData.name}</Text>
						<Text>邮箱：{formData.email}</Text>
						<Text dimColor>按 Enter 完成，Esc 返回</Text>
					</Box>
				)}
			</Step>
		</Stepper>
	);
};
```

### 5. 其他有用的 Ink 组件

| 组件 | 功能 | 安装 |
|------|------|------|
| [ink-link](https://github.com/sindresorhus/ink-link) | 可点击的链接 | `npm install ink-link` |
| [ink-gradient](https://github.com/sindresorhus/ink-gradient) | 渐变色文字 | `npm install ink-gradient` |
| [ink-big-text](https://github.com/sindresorhus/ink-big-text) | ASCII 艺术字 | `npm install ink-big-text` |
| [ink-progress-bar](https://github.com/brigand/ink-progress-bar) | 进度条 | `npm install ink-progress-bar` |
| [ink-table](https://github.com/maticzav/ink-table) | 表格组件 | `npm install ink-table` |
| [ink-form](https://github.com/lukasbach/ink-form) | 表单验证 | `npm install ink-form` |
| [ink-markdown](https://github.com/cameronhunter/ink-markdown) | Markdown 渲染 | `npm install ink-markdown` |
| [ink-syntax-highlight](https://github.com/vsashyn/ink-syntax-highlight) | 代码高亮 | `npm install ink-syntax-highlight` |
| [ink-task-list](https://github.com/privatenumber/ink-task-list) | 任务列表 | `npm install ink-task-list` |

## 实战项目：构建交互式 CLI 应用

让我们构建一个完整的任务管理 CLI 应用。

### 项目结构

```
task-manager/
├── src/
│   ├── cli.ts              # 入口文件
│   ├── app.tsx             # 主应用
│   └── components/
│       ├── dashboard.tsx   # 任务面板
│       ├── add-task.tsx    # 添加任务
│       └── task-list.tsx   # 任务列表
├── package.json
└── tsconfig.json
```

### 完整代码

`src/cli.ts`：

```typescript
#!/usr/bin/env node
import React from 'react';
import {render} from 'ink';
import App from './app.js';

render(React.createElement(App));
```

`src/app.tsx`：

```typescript
import React, {useState} from 'react';
import {Box, Text, useInput, useApp} from 'ink';
import Dashboard from './components/dashboard.js';
import TaskList from './components/task-list.js';
import AddTask from './components/add-task.js';

interface Task {
	id: number;
	title: string;
	completed: boolean;
}

const App: React.FC = () => {
	const {exit} = useApp();
	const [page, setPage] = useState<'dashboard' | 'list' | 'add'>('dashboard');
	const [tasks, setTasks] = useState<Task[]>([
		{id: 1, title: '学习 Ink', completed: true},
		{id: 2, title: '构建 CLI 应用', completed: false},
		{id: 3, title: '发布到 npm', completed: false},
	]);

	useInput((input, key) => {
		if (input === 'q') {
			exit();
		}

		if (key.escape) {
			setPage('dashboard');
		}

		if (input === '1') setPage('dashboard');
		if (input === '2') setPage('list');
		if (input === '3') setPage('add');
	});

	const addTask = (title: string) => {
		const newTask: Task = {
			id: tasks.length + 1,
			title,
			completed: false,
		};
		setTasks([...tasks, newTask]);
		setPage('list');
	};

	const toggleTask = (id: number) => {
		setTasks(tasks.map(t => 
			t.id === id ? {...t, completed: !t.completed} : t
		));
	};

	const renderPage = () => {
		switch (page) {
			case 'dashboard':
				return <Dashboard tasks={tasks} />;
			case 'list':
				return <TaskList tasks={tasks} onToggle={toggleTask} />;
			case 'add':
				return <AddTask onAdd={addTask} />;
			default:
				return null;
		}
	};

	return (
		<Box flexDirection="column" padding={1}>
			{/* 头部 */}
			<Box flexDirection="column" marginBottom={1}>
				<Text bold color="cyan">╔════════════════════════════════╗</Text>
				<Text bold color="cyan">║   📋 Task Manager - Ink Demo  ║</Text>
				<Text bold color="cyan">╚════════════════════════════════╝</Text>
			</Box>

			{/* 导航 */}
			<Box marginBottom={1}>
				<Text color={page === 'dashboard' ? 'green' : 'gray'}>
					{page === 'dashboard' ? '▶ ' : '  '}[1] 面板
				</Text>
				<Text color={page === 'list' ? 'green' : 'gray'}>
					{page === 'list' ? ' ▶ ' : '  '}[2] 任务列表
				</Text>
				<Text color={page === 'add' ? 'green' : 'gray'}>
					{page === 'add' ? ' ▶ ' : '  '}[3] 添加任务
				</Text>
			</Box>

			{/* 内容 */}
			{renderPage()}

			{/* 底部提示 */}
			<Box marginTop={1}>
				<Text dimColor>1/2/3 切换页面 | Esc 返回主页 | q 退出</Text>
			</Box>
		</Box>
	);
};

export default App;
```

`src/components/dashboard.tsx`：

```typescript
import React from 'react';
import {Box, Text} from 'ink';

interface Task {
	id: number;
	title: string;
	completed: boolean;
}

interface Props {
	tasks: Task[];
}

const Dashboard: React.FC<Props> = ({tasks}) => {
	const total = tasks.length;
	const completed = tasks.filter(t => t.completed).length;
	const pending = total - completed;
	const percentage = total > 0 ? Math.round((completed / total) * 100) : 0;

	const progressBar = '█'.repeat(Math.floor(percentage / 3.33)) + 
	                    '░'.repeat(30 - Math.floor(percentage / 3.33));

	return (
		<Box flexDirection="column">
			<Text bold color="yellow">📊 任务统计</Text>
			
			<Box flexDirection="column" marginTop={1}>
				<Text>总任务数：{total}</Text>
				<Text color="green">已完成：{completed}</Text>
				<Text color="cyan">待完成：{pending}</Text>
			</Box>

			<Box marginTop={1}>
				<Text color="blue">[{progressBar}] {percentage}%</Text>
			</Box>
		</Box>
	);
};

export default Dashboard;
```

`src/components/task-list.tsx`：

```typescript
import React, {useState} from 'react';
import {Box, Text} from 'ink';

interface Task {
	id: number;
	title: string;
	completed: boolean;
}

interface Props {
	tasks: Task[];
	onToggle: (id: number) => void;
}

const TaskList: React.FC<Props> = ({tasks, onToggle}) => {
	const [selectedIndex, setSelectedIndex] = useState(0);

	const handleInput = (input: string, key: any) => {
		if (key.upArrow) {
			setSelectedIndex(prev => Math.max(0, prev - 1));
		}
		if (key.downArrow) {
			setSelectedIndex(prev => Math.min(tasks.length - 1, prev + 1));
		}
		if (key.return) {
			onToggle(tasks[selectedIndex].id);
		}
	};

	return (
		<Box flexDirection="column">
			<Text bold color="magenta">📝 任务列表</Text>

			{tasks.map((task, index) => (
				<Box key={task.id}>
					<Text color={index === selectedIndex ? 'green' : 'white'}>
						{index === selectedIndex ? '▶ ' : '  '}
						{task.completed ? '✅' : '⬜'} {task.title}
					</Text>
				</Box>
			))}

			<Box marginTop={1}>
				<Text dimColor>↑↓ 选择 | Enter 切换状态</Text>
			</Box>
		</Box>
	);
};

export default TaskList;
```

`src/components/add-task.tsx`：

```typescript
import React, {useState} from 'react';
import {Box, Text} from 'ink';
import TextInput from 'ink-text-input';

interface Props {
	onAdd: (title: string) => void;
}

const AddTask: React.FC<Props> = ({onAdd}) => {
	const [value, setValue] = useState('');

	const handleSubmit = (value: string) => {
		if (value.trim()) {
			onAdd(value);
		}
	};

	return (
		<Box flexDirection="column">
			<Text bold color="green">➕ 添加新任务</Text>

			<Box marginTop={1}>
				<Text color="cyan">$ </Text>
				<TextInput
					value={value}
					onChange={setValue}
					onSubmit={handleSubmit}
					placeholder="输入任务名称..."
				/>
			</Box>
		</Box>
	);
};

export default AddTask;
```

## 测试 Ink 组件

使用 `ink-testing-library` 进行简单测试：

```bash
npm install -D ink-testing-library
```

```typescript
import React from 'react';
import {render} from 'ink-testing-library';
import {Text} from 'ink';

test('渲染文字', () => {
	const {lastFrame} = render(<Text>Hello World</Text>);
	
	expect(lastFrame()).toContain('Hello World');
});
```

## 最佳实践

### 1. 性能优化

```typescript
// ✅ 好的做法：使用 React.memo
const ExpensiveComponent = React.memo(({data}) => {
	return <Text>{data}</Text>;
});

// ✅ 好的做法：合理使用 key
{items.map(item => (
	<Text key={item.id}>{item.name}</Text>
))}

// ❌ 避免：在渲染函数中创建新对象
{items.map((item, index) => (
	<Text key={index}>{item.name}</Text>  // 不要用 index 作为 key
))}
```

### 2. 错误处理

```typescript
import React, {Component} from 'react';
import {Box, Text} from 'ink';

class ErrorBoundary extends Component<{children: React.ReactNode}> {
	state = {hasError: false};

	static getDerivedStateFromError() {
		return {hasError: true};
	}

	render() {
		if (this.state.hasError) {
			return <Text color="red">❌ 出错了！</Text>;
		}

		return this.props.children;
	}
}

// 使用
const App = () => (
	<ErrorBoundary>
		<YourComponent />
	</ErrorBoundary>
);
```

### 3. 协调键盘输入

当组件中有多个输入源时，需要协调：

```typescript
// 使用上下文管理焦点
const FocusContext = React.createContext(0);

const App = () => {
	const [focusedIndex, setFocusedIndex] = useState(0);

	return (
		<FocusContext.Provider value={focusedIndex}>
			<MenuList />
		</FocusContext.Provider>
	);
};
```

### 4. 响应式布局

```typescript
import {useStdout} from 'ink';

const ResponsiveComponent = () => {
	const {stdout} = useStdout();
	const width = stdout?.columns || 80;

	return (
		<Text>
			终端宽度：{width} 字符
			{width > 60 ? ' (宽屏)' : ' (窄屏)'}
		</Text>
	);
};
```

## 调试技巧

### 1. 使用 React DevTools

```bash
# 安装 react-devtools-core
npm install -D react-devtools-core

# 启动应用（设置环境变量）
DEV=true npm start

# 在另一个终端启动 DevTools
npx react-devtools
```

### 2. 字符串渲染测试

```typescript
import {renderToString} from 'ink';

const output = renderToString(
	<Box padding={1}>
		<Text color="green">Hello</Text>
	</Box>
);

console.log(output);
```

## 总结

Ink 为 CLI 开发带来了全新的范式：

✅ **组件化**：可复用的 UI 组件
✅ **声明式**：用 JSX 描述界面
✅ **生态丰富**：大量现成组件可用
✅ **类型安全**：完整的 TypeScript 支持
✅ **易于测试**：专门的测试库

### 适用场景

- 交互式 CLI 工具
- 配置向导
- 数据监控面板
- 任务管理器
- 游戏（是的，可以用 Ink 做游戏！）

### 学习资源

- [Ink 官方文档](https://github.com/vadimdemedes/ink)
- [Ink 示例](https://github.com/vadimdemedes/ink/tree/master/examples)
- [freeCodeCamp 教程](https://www.freecodecamp.org/react-js-ink-cli-tutorial/)

希望这篇指南能帮你快速入门 Ink！如果你有任何问题，欢迎在评论区留言。
