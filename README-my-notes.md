# notes for upgrade

## missing js files from dist folder
If after update there errors for missings ***.min.js*** files, then run the following command to generate them:
```bash
npm install
npm run build

# Then, commit the changes and push them to the remote repository.
git add assets/js/dist _sass/vendors -f
git commit -m "update js files"

```
