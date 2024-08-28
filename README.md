# Usage

`composer cs-fix`

-  run inside pod
-  run composer update beforehand to make sure you have the most recent version
-  check changes (outside pod) with `git diff` and commit

Useful `alias` for csfix in plugins:
```
alias plugin-csfix='POD=$(kubectl get pods -n krn-dev -l krn=backend -o custom-columns=:metadata.name); kubectl -n krn-dev exec -c backend $POD -- /bin/bash -c "cd $(pwd) && composer update -q --no-plugins --ignore-platform-reqs && composer cs-fix -q --no-plugins"'
```
(Run this command *in the plugin-directory on your VM.* It will exec into backend-pod, go to the same plugin-directory inside the pod and run the correct commands there for you.)


<br />

# How-To: fix csfix deprecation errors / update csfix rules

If `csfix` is running fine on your VM with no diff output, but the pipelines on GitLab fail, there is most likely a new version running in the pipeline with updated rulesets

### Preparation

- exec into the respective pod
- got to the repo's directory inside the pod
- run composer update
- run chown on vendor-directory

example for a plugin:
```sh
'POD=$(kubectl get pods -n krn-dev -l krn=backend -o custom-columns=:metadata.name); kubectl -n krn-dev exec -c backend $POD -- /bin/bash
cd /WordPress/plugins/kmm-article
composer update -q --no-plugins --ignore-platform-reqs
chown -R deploy:deploy vendor
```

<br />

### What to change

- open the vendor-directory of the repository in VSCode
- open `vendor/krn/php-cs-style/src/Php73.php`
- look into the failed pipeline's output und search for the rule you want to change/fix
- look up the rule here: https://cs.symfony.com/doc/rules/ and open the page
- if the rule is deprecated, you will see a link to the new rule that replaces it
- **read the new rule's documentation and check how to configure it so it works the same / similar enough to the old one** 
- replace the broken rule with the new one
- run csfix in the pod to verify your changes are working correctly

<br />

### Creating a Merge Request
**You don't need to checkout the PHPCS repo locally to commit your changes** 
- go to the repo on Gitlab
- click "Edit"-Button -> "WebIDE"
- copy and paste the changed contents of `Php73.php`
- commit to a new branch and create a merge request, all right in the browser