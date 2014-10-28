## Contribute to the Platform.sh documentation

Here is what you need to build the Platform.sh documentation locally.

### Getting Started

1. Clone the repository

  ```
  $ git clone git@github.com:platformsh/platformsh-docs.git
  $ cd platformsh-docs
  ```

2. Install [Gitbook](https://github.com/GitbookIO/gitbook) and the required plugins via NPM

  ```
  $ npm install gitbook -g
  $ npm install gitbook-plugin-richquotes
  $ npm install --save gitbook-plugin-anchors
  ```

3. Build the book

  ```
  gitbook build
  ```

Once the build is finished, the HTML pages should be in `_book/`.

### Using the Gitbook Editor

You can download the Gitbook Editor [here](https://www.gitbook.io/#write).


![http://creativecommons.org/licenses/by-sa/4.0/](images/CC-BY-SA.png)
