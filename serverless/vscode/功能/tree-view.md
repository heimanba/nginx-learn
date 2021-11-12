### 基本实现
#### TreeDataProvider生成树视图所需依赖数据
- `getChildren(element?: T): ProviderResult<T[]>`返回指定节点
- `getTreeItem(element: T): TreeItem | Thenable<TreeItem>`用于视图展示的UI节点
- `TreeItemCollapsibleState`折叠事件
	- Collapsed 折叠
	- Expanded 展开
	- None 无节点，不会触发折叠状态

#### package.json

```
"activationEvents": [
    "onView:nodeDependencies"
],
"contributes": {
	"viewsContainers": {
      "activitybar": [
        {
          "id": "Serverless-Devs-explorer",
          "title": "Serverless-Devs",
          "icon": "media/dep.svg"
        }
      ]
    },
    "views": {
      "Serverless-Devs-explorer": [
        {
          "id": "nodeDependencies",
          "name": "Node Dependencies"
        }
      ]
    }
},
```

#### extension.ts

```
import * as vscode from 'vscode';
import { DepNodeProvider } from './nodeDependencies';

const rootPath = (vscode.workspace.workspaceFolders && (vscode.workspace.workspaceFolders.length > 0))
		? vscode.workspace.workspaceFolders[0].uri.fsPath : undefined;

// Samples of `window.registerTreeDataProvider`
const nodeDependenciesProvider = new DepNodeProvider(rootPath);
vscode.window.registerTreeDataProvider('nodeDependencies', nodeDependenciesProvider);
```

- nodeDependencies.ts

```
import * as vscode from 'vscode';
import * as fs from 'fs';
import * as path from 'path';

export class DepNodeProvider implements vscode.TreeDataProvider<Dependency> {

	private _onDidChangeTreeData: vscode.EventEmitter<Dependency | undefined | void> = new vscode.EventEmitter<Dependency | undefined | void>();
	readonly onDidChangeTreeData: vscode.Event<Dependency | undefined | void> = this._onDidChangeTreeData.event;

	constructor(private workspaceRoot: string | undefined) {
	}

	refresh(): void {
		this._onDidChangeTreeData.fire();
	}

	getTreeItem(element: Dependency): vscode.TreeItem {
		return element;
	}

	getChildren(element?: Dependency): Thenable<Dependency[]> {
		if (!this.workspaceRoot) {
			vscode.window.showInformationMessage('No dependency in empty workspace');
			return Promise.resolve([]);
		}

		if (element) {
			return Promise.resolve(this.getDepsInPackageJson(path.join(this.workspaceRoot, 'node_modules', element.label, 'package.json')));
		} else {
			const packageJsonPath = path.join(this.workspaceRoot, 'package.json');
			if (this.pathExists(packageJsonPath)) {
				return Promise.resolve(this.getDepsInPackageJson(packageJsonPath));
			} else {
				vscode.window.showInformationMessage('Workspace has no package.json');
				return Promise.resolve([]);
			}
		}

	}

	/**
	 * Given the path to package.json, read all its dependencies and devDependencies.
	 */
	private getDepsInPackageJson(packageJsonPath: string): Dependency[] {
		if (this.pathExists(packageJsonPath)) {
			const packageJson = JSON.parse(fs.readFileSync(packageJsonPath, 'utf-8'));

			const toDep = (moduleName: string, version: string): Dependency => {
				// @ts-ignore
				if (this.pathExists(path.join(this.workspaceRoot, 'node_modules', moduleName))) {
					return new Dependency(moduleName, version, vscode.TreeItemCollapsibleState.Collapsed);
				} else {
					return new Dependency(moduleName, version, vscode.TreeItemCollapsibleState.None, {
						command: 'extension.openPackageOnNpm',
						title: '',
						arguments: [moduleName]
					});
				}
			};

			const deps = packageJson.dependencies
				? Object.keys(packageJson.dependencies).map(dep => toDep(dep, packageJson.dependencies[dep]))
				: [];
			const devDeps = packageJson.devDependencies
				? Object.keys(packageJson.devDependencies).map(dep => toDep(dep, packageJson.devDependencies[dep]))
				: [];
			return deps.concat(devDeps);
		} else {
			return [];
		}
	}

	private pathExists(p: string): boolean {
		try {
			fs.accessSync(p);
		} catch (err) {
			return false;
		}

		return true;
	}
}

export class Dependency extends vscode.TreeItem {

	constructor(
		public readonly label: string,
		private readonly version: string,
		public readonly collapsibleState: vscode.TreeItemCollapsibleState,
		public readonly command?: vscode.Command
	) {
		super(label, collapsibleState);

		this.tooltip = `${this.label}-${this.version}`;
		this.description = this.version;
	}

	iconPath = {
		light: path.join(__filename, '..', '..', 'resources', 'light', 'dependency.svg'),
		dark: path.join(__filename, '..', '..', 'resources', 'dark', 'dependency.svg')
	};

	contextValue = 'dependency';
}
```

![](https://img.alicdn.com/imgextra/i1/O1CN0186eue9200xCp4GFe5_!!6000000006788-2-tps-1418-1170.png)

### 添加事件
- package.json
```
"activationEvents": [
    "onView:nodeDependencies"
],
"contributes": {
    "menus": {
      "view/title": [
        {
          "command": "nodeDependencies.refreshEntry",
          "when": "view == nodeDependencies",
          "group": "navigation"
        }
      ]
    },
    "commands": [
      {
        "command": "nodeDependencies.refreshEntry",
        "title": "Refresh",
        "icon": {
          "light": "resources/light/refresh.svg",
          "dark": "resources/dark/refresh.svg"
        }
      }
    ],
    "views": {
      "explorer": [
        {
          "id": "nodeDependencies",
          "name": "Node Dependencies"
        }
      ]
    }
}
```
- extension.ts
```
vscode.commands.registerCommand('nodeDependencies.refreshEntry', () =>
    nodeDependenciesProvider.refresh()
);
```
![](https://img.alicdn.com/imgextra/i3/O1CN01AArlv11Ceah6myUer_!!6000000000106-2-tps-476-712.png)


### View Container
#### activitybar
```
"activationEvents": [
    "onView:nodeDependencies"
],
"views": {
    "package-explorer": [
    {
        "id": "nodeDependencies",
        "name": "Node Dependencies",
        "icon": "media/dep.svg",
        "contextualTitle": "Package Explorer"
    }
    ]
},
"viewsContainers": {
    "activitybar": [
        {
            "id": "package-explorer",
            "title": "Package Explorer",
            "icon": "media/dep.svg"
        }
    ]
}
```
![](https://img.alicdn.com/imgextra/i2/O1CN015Poedd1yJ9iTgp3XS_!!6000000006557-2-tps-684-1084.png)

#### panel
```
"activationEvents": [
    "onView:nodeDependencies"
],
"views": {
    "package-explorer": [
    {
        "id": "nodeDependencies",
        "name": "Node Dependencies",
        "icon": "media/dep.svg",
        "contextualTitle": "Package Explorer"
    }
    ]
},
"viewsContainers": {
    "panel": [
        {
          "id": "package-explorer",
          "title": "Package Explorer",
          "icon": "media/dep.svg"
        }
    ]
}
```

![](https://img.alicdn.com/imgextra/i4/O1CN01wTOu1u1eLB9PHk6PY_!!6000000003854-2-tps-1972-484.png)

