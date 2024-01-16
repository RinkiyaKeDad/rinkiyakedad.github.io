# Hugo Instructions

### Cloning And Updating

1. Theme being used: [PaperMod](https://github.com/adityatelange/hugo-PaperMod)

2. Take care while [cloning](https://github.com/adityatelange/hugo-PaperMod/wiki/Installation):
```
git submodule update --init --recursive # needed when you reclone your repo (submodules may not get cloned automatically)
```

3. To [update](https://github.com/adityatelange/hugo-PaperMod/wiki/Installation) the theme:
```
git submodule update --remote --merge
```

### Deploying

1. `git submodule add -b master https://github.com/RinkiyaKeDad/rinkiyakedad.github.io.git public` (first time only)

2. `hugo` in the root of this repo to geenrate the files in public directory.

3. `cd public` and then commit and push. To confirm `cd public` and do `git remote -v`.

4. The commit and push from root of this repo so that it updates the version of `public` it has. (`git push origin HEAD:master)`

5. `cd ..` and commit and push from the root of this repo. 

### Resources

1. [This](https://youtu.be/LIFvgrRxdt4) video.
2. To update the theme: `git submodule update --remote --merge`