## A collection of short bash scripts to make life a little easier

Pull requests and additions are most welcome!

### Lint only the files changed in the current commit

This is a cool one-liner that does a `git diff` to get a list of files changed and only lints these files!

This example uses `eslint` for linting:

```js
"LIST=`git diff-index --name-only HEAD | grep .*\\.js | grep -v json`; if [ \"$LIST\" ]; then eslint $LIST; fi"
```
Add it as a script to your package.json and add it to the pre-commit hook!


```js
"scripts": {
  "lint": "LIST=`git diff-index --name-only HEAD | grep .*\\.js | grep -v json`; if [ \"$LIST\" ]; then eslint $LIST; fi",
},
"pre-commit": [
  "lint"
],
```


### Generate a template react-native component

This is a useful script to avoid having to write out all the boilerplate code for setting up a new component.  It can be used to create a new react-native component with a specified name in a specified output folder.

Save the script in your project folder and call it something like `react-native-component-init.sh`.

Run the script in the terminal by typing `bash react-native-component-init.sh`. Or turn the script into an executable by typing `chmod 755 <name of file> `.

You can also add a script command to your package.json to run the script e.g.

```js
"scripts": {
  "component-init": "./react-native-component-init.sh",
}
```

Then simply call it by `npm run component-init`

```bash
#!/bin/bash

echo -n 'Enter the name of the Component: '
read ComponentName
echo -n 'Output folder path relative to ./app: '
read Output

ABS_PATH=`cd "$1"; pwd` # gets the absolute path of the directory in which the script is saved

if [ -e "$ABS_PATH/app/$Output/$ComponentName.js" ]
  then echo "File named $ComponentName already exists in the '$ABS_PATH/app/$Output/' folder"
else
  echo "
'use strict';

import React, { Component, StyleSheet, PropTypes, View } from 'react-native';

class $ComponentName extends Component {

  constructor(props) {
    super(props);
  }

  render(){
    return (
      <View style = { styles.container } >
      </View>
    )
  }

}

$ComponentName.propTypes = {

}

const styles = StyleSheet.create({
  container: {

  }
})

export default $ComponentName;

" > "$ABS_PATH/app/$Output/$ComponentName.js" # path of the new component - modify this as necessary

  open "$ABS_PATH/app/$Output/$ComponentName.js"
fi

```

The best part is that the template can be modified to generate a React Component, html page or any boilerplate code that needs to be repeated multiple times!


## Git Hook to automatically run `npm install` if the package.json has changed

First set up global configuration for git hooks: In your home directory, create a `.git_template` folder with a `hooks` folder inside.
``` 
mkdir -p ~/.git_template/hooks
```

Add a file called 'post-merge' with the following code:

```bash
#/usr/bin/env bash
# MIT © Sindre Sorhus - sindresorhus.com
# forked by Gianluca Guarini
# forked by Nikhila Ravi

changed_files="$(git diff-tree -r --name-only --no-commit-id ORIG_HEAD HEAD)"

check_run() {
	echo "$changed_files" | grep --quiet "$1" && eval "$2"
}

# run `npm install` and `npm prune` if the `package.json` file gets changed
# updates all the npm dependencies and deletes the unused packages (not listed into the  `package.json` file)

check_run package.json "npm install && npm prune"
```

Make the file executable by running this command in the terminal:
```
chmod a+x ~/.git-templates/hooks/post-merge
```

Set your git config to add the git hook when a repo is initialised :  
```
git config --global init.templatedir '~/.git_template'
```

All new repos that are cloned will have the hooks added to the .git folder.

For existing repos run `git init` inside the repo and the hook will be added to the .git folder (existing hooks will not be overwritten).

Note: This hook won't be executed if the merge failed due to conflicts.
