You get the message
```sh
undefined
```

To **avoid** this all you have to do is install a package directly into your node session by typing the command:

```sh
node -e "require('repl').start({ignoreUndefined: true})"
```

Now you will be able to type into the REPL prompt without being annoyed by undefined warnings every time youy press return.

You can see my original question on StackOverflow where I learned this method here:

http://stackoverflow.com/questions/34972737/how-to-avoid-undefined-output-from-node-repl

(feel free to up-vote this question as it already has at least one downvote on it for apparently being too simple for the SO community)