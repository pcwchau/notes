# Index

- [Index](#index)
- [To Explore](#to-explore)
- [Tools](#tools)
  - [Develop tools](#develop-tools)
  - [Management tools](#management-tools)
  - [Visual Studio](#visual-studio)
  - [VS Code](#vs-code)
    - [Keyboard Shortcuts](#keyboard-shortcuts)
  - [IntelliJ](#intellij)
  - [Google chrome](#google-chrome)
  - [Notepad++](#notepad)
  - [Coding Practice](#coding-practice)
    - [General](#general)
      - [Separation of Code and Configuration](#separation-of-code-and-configuration)
      - [Error Handling](#error-handling)
    - [OOP](#oop)
    - [Version Control](#version-control)
    - [Network](#network)

# To Explore

- [Stack Overflow Annual Developer Survey](https://insights.stackoverflow.com/survey)
- https://www.tiobe.com/tiobe-index/
- lazy loading
- infrastructure as code, make data easier to view
- difference between opensearch, mongo, redis, how to do searching as fast as google does, if so still need caching?
- how to do auth? token / session + cookie

# Tools

## Develop tools

- Docker: set up a local container and sync environment between developers
- Fiddler: ip address mapping (sometimes may cause SSL problem)
- Postman: testing API (GET/POST/... requests), export `curl` command, batch export API
- Sourcetree: check git branch tree
- Snipaste: `F1` screenshot, `F3` pin to screen, `Alt+1` screenshot and copy
- Ditto clipboard manager: clipboard history, `Alt+2` open clipboard
- VS Code
- Java
  - IntelliJ
  - java jdk
  - maven
- JS
  - node.js
- Notepad++
- Cmder: run linux and git in windows (can integrate with windows terminal!!)
- DB admin tools:
  - HeidiSQL (can gen alter code)
  - dbeaver (DB in SSH tunnel + jump server, set advanced setting use JSch, driver allow publickeyretrieval, ER diagram)
  - MySQL workbench (ER diagram)
  - SQL Server Management Studio (for MSSQL server)
- Git
- Tortoise SVN
- Chocolatey - a popular package manager for Windows.
- sonarqube - auto check bugs and security issues.

## Management tools

- Thunderbird: email client with filter
- Slack / discord: communication
- Asana (web): see everyone schedule, project timeline
- Jira (web): task description and distribution (can use robot to sync with gitlab and slack)
- Gitlab (git): more private version of Git

## Visual Studio
Preference:
- Set indentations to `tab` instead of `space`
- Set class template when creating a new class file in `C:\Program Files\Microsoft Visual Studio\2022\Community\Common7\IDE\ItemTemplates\CSharp\Code\1033\Class\Class.cs`
- Set autosave - Use Visual Studio Search (Ctrl+Q) and look for “autosave”. If “Automatically save files when Visual Studio is in the background” is checked, any time Visual Studio loses focus, usually as you change to another application in Windows, VS will attempt to save every dirty document in the IDE.

Extensions:
- `CodeMaid` - spade = outline
- `Viasfora` - colour brackets, long press `ctrl` will have highlight

## VS Code

Preference (user setting):

- Enable/Disable preview mode (always open file in new tab)
  - `workbench.editor.enablePreview`
- Enable/Disable minimap (right bar to see the code)
  - `editor.minimap.enabled`
- Save file automatically
  - `files.autoSave` -> onFocusChange
- Enable multiple lines of tabs
  - `workbench.editor.showTabs` -> multiple, then enable `workbench.editor.wrapTabs`

Extensions: 

- `GitLens` - see git commit/author for each line
- `Indent one space` - can use space to indent
- `indent-rainbow` - makes indentation easier to look
- `Markdown all in one` - for markdown
- `Markdown Table Formatter` - for markdown table format
- `Markdown Preview Github Styling` - for markdown preview style
- `React Hooks Snippets` - add snippets for hooks like `useState` and `useEffect`
- `Copy Json Path` - very useful for language file in react
- `ESLint` - syntax / potential runtime error checking for JS, JSX, TS and TSX.
- `Prettier - Code Formatter` - apply formatting to JS, JSX, TS and TSX.
  - You can set the config for each project in `package.json` or `config.file`. (when working with JS/TS, better use tab size = 2)
  ```json
  // package.json
  "prettier": {
    "printWidth": 120
  }
  ```
  - For some cases where you don't want to auto format, use "save without formatting".
  ```json
  // In your workspace .vscode/settings.json (open user setting -> open settings JSON)
  {
    // ... some other configs made by you
    // autoformatter
    "[typescript]": {
      "editor.formatOnSave": true,
      "editor.tabSize": 2,
      "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "[typescriptreact]": {
      "editor.formatOnSave": true,
      "editor.tabSize": 2,
      "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "[javascript]": {
      "editor.formatOnSave": true,
      "editor.tabSize": 2,
      "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "[javascriptreact]": {
      "editor.formatOnSave": true,
      "editor.tabSize": 2,
      "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "[css]": {
      "editor.formatOnSave": true,
      "editor.tabSize": 2,
      "editor.defaultFormatter": "esbenp.prettier-vscode"
    }
  }
  ```

### Keyboard Shortcuts

Keymap | Function
------ | --------
`Ctrl` + `,` | open user setting

Searching

| Keymap                 | Function               |
|------------------------|------------------------|
| `Ctrl` + `F`           | Search in current file |
| `Ctrl` + `Shift` + `F` | Search in all files    |
| `Ctrl` + `P`           | Search file            |

Navigation

| Keymap                        | Function                                          |
|-------------------------------|---------------------------------------------------|
| `Alt` + `left`                | Navigate back                                     |
| `Alt` + `right`               | Navigate forward                                  |
| `Ctrl` + `left click` / `F12` | Go to declaration or usage                        |
| `Alt` + `left click`          | Go to implementation (useful for abstract method) |

Editor

| Keymap                     | Function                               |
|----------------------------|----------------------------------------|
| `Alt` + `up`               | move statement up                      |
| `Alt` + `down`             | move statement down                    |
| `Ctrl` + `Alt` + `up`      | add cursor above (edit multiple lines) |
| `Ctrl` + `Alt` + `down`    | add cursor down (edit multiple lines)  |
| `Shift` + `Alt` + `down`   | copy one line down                     |
| `Ctrl` + `/`               | comment with line                      |
| `Ctrl` + `D`               | delete line                            |
| `Ctrl` + `enter`           | open new line                          |
| `Ctrl` + `K`, `Ctrl` + `0` | Fold all (collapse all)                |
| `Ctrl` + `K`, `Ctrl` + `1` | Fold level 1                           |
| `Ctrl` + `K`, `Ctrl` + `J` | Unfold all                             |

Debug

| Keymap | Function       |
|--------|----------------|
| `F2`   | set value      |
| `F10`  | resume program |
| `F11`  | step over      |

Other

| Keymap                         | Function                                               |
|--------------------------------|--------------------------------------------------------|
| `Alt` + `Enter`                | Show recommended code action                           |
| `Alt` + `A`                    | check who commit that line (annotate with git blame)   |
| `Alt` + `Shift` + `F`          | reformat code                                          |
| `Ctrl` + `Alt` + `Shift` + `L` | open reformat code dialog (eg. optimize import or not) |
| `F2`                           | rename                                                 |
| `Ctrl` + `                     | open run / terminal                                    |

## IntelliJ
Set up:
- Project SDK: File -> project structure -> project
- Language level: define coding assistance features that the editor provides
- **Example for java level**: If you use java 9 feature, your SDK must be higher than java 9 and the language level should be higher than java 9 (but it would not affect compile/build).
- **Tips for Maven project**: The `<source>` and `<target>` setting in `pom.xml` does not guarantee the java version. Only JDK version can guarantee the java version.

Plugin:
- Rainbow bracket
- String Manipulation
- Archive browser
- google-java-format (For Webflux / Reactor / RxJava Projects only)
- one dark theme

## Google chrome
Extension:
- JSON formatter
- Allow CORS
- TamperMonkey (auto run JS in browser console)
- React developer tools
- Redux DevTools 

## Notepad++
Plugin:
- SQLinForm: SQL formatter
- JStool: json formatter
- Compare plus
  - It can compare two opened files in real time.

## Coding Practice

- Twelve Factors App: https://12factor.net/

### General

#### Separation of Code and Configuration

Use configuration data to change the behavior of a program without having to modify its source code. The configuration data can be stored in a config file  or in a database.

For example, configuration data can be:

- Timeout setting
- Database path
- External API URL

To separate config files in different environment:

- Method 1: with K8S and .gitlab-ci.yml, put different config folder in ./manifest
- Method 2: put all config folders in ./src/main/resources, in `pom.xml` define different profiles, then use `mvn install -P{profile}`

#### Error Handling

- try / catch
- DB need rollback?
- Retry after an interval, suitable for:
  - Some DB operation that must not have errors except connection problem.
- Timeout of an operation
- Ensure the operation is finished: update DB, then use DB value to update object.

Be aware of how to debug
- Add log, the format can be `method_name - description_with_parameter - [status] - reason`
  - `status` can be `START`, `SUCCESS`, `FAIL`, `END`, etc.
- For web application, it is important to tell the client if a system error happens. Do not just return empty result.

Be aware of how to format, one line of code should not be too long (you should able to view two files in horizontal, good for compare in git, review history, etc.)

Be aware of the readability, write summary / java doc.

### OOP

- Static class / method: Static classes are useful and provide an easy way to access its members that does not need to work differently for different objects. Other than that, use OOP for easier maintainence.
- Each class should follow **single responsibility principle (SRP)**. Remember to separate different service into different classes. **Don't always fix problem by injecting dependency.**
- Dependency injection
  - In order to do **dependency injection**, you can use constructor injection (if very obvious construction relationship, i.e. need to construct A with B, but no need construct B with A) or other similar things, such as C# property injection. If you really have to deal with *circular dependency* (which also is not good), you may use a method, instead of constructor, to initialize the class.
  - Think clearly about the construction relationship / sequence before development.

### Version Control

- For every commit, it should cover only the change of the features mentioned in the comment. It is easier to revert in the future.
- Before commit, use `Sourcetree` check every file to ensure the change is correct
- If fallback before, remember to build back later even no code changes (i.e. dont build backend only but forget frontend)

### Network

- Send only required information to frontend. For example, only a field in the user object is changed, then just send this field instead of whole user object.
- Handle disconnection if your application would connect to an external server, such as a gateway, a database server, etc. 
  - How to reconnect automatically?
  - If reconnect, need to get all data again for all user at the same time? Will the data that needs to update after reconnect use the same method as initialization?
