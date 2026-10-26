Node.js - v22.22.3 or newer
Text editor - We recommend Visual Studio Code
Terminal - Required for running Angular CLI commands
Development Tool - To improve your development workflow, we recommend the Angular Language Service

Install Angular CLI
=> npm install -g @angular/cli

Create a new project
=> ng new <project-name>
new frontend_angular --routing --style=css --ssr=false --strict --ai-config=none
=> cd my-first-angular-app
=> npm start

ng generate component features/dashboard
ng generate component features/groups
ng generate component features/requests

To enable the latest Language Service features, set the strictTemplates option in tsconfig.json by setting strictTemplates to true, as shown in the following example:

"angularCompilerOptions": {
"strictTemplates": true
}

Install the latest version of TypeScript in a local node_modules directory:
=> npm install --save-dev typescript
Install the Angular Language Service package in the same location:
=> npm install --save-dev @angular/language-service

Once the package is installed, add the following to the "compilerOptions" section of your project's tsconfig.json.

tsconfig.json
"plugins": [
{"name": "@angular/language-service"}
]

In your editor's user preferences (Cmd+, or Ctrl+,), add the following:

Sublime Text user preferences
"typescript-tsdk": "<path to your folder>/node_modules/typescript/lib"
