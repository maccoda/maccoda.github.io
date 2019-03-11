+++
date = 2019-03-11
title = "Managing Local Heroku Deployments"
description = "Likely overkill to ensure you know what you have deployed"
+++

Recently I have using [Heroku] a fair bit on a hobby project I am working on and
having a hard time getting comfortable with local deployments. As my project is
starting to get a little more complex and actually useful I am wanting to make
sure what I have deployed is something that works. Further to it I have a back end
deployed separately from the front end application so I want to make sure they
are in sync.

All of this lands me at a point where I would love to build in a proper CI/CD
pipeline for these projects, however I struggle to bring myself to it as I am
lazy and couldn't find a short solution in the first 3.5 seconds
:stuck_out_tongue_closed_eyes:. So I went for the next best thing, an
over-engineered script that gives me some certainty in what I am releasing. This
is what I wanted to share so hopefully someone who gets into the same position
as myself has something pre-made for them.

```bash
#!/usr/bin/env bash
set -e

echo "Last release:"
cat .last-release
echo
stats=$(git status --porcelain)
if [[ -z $stats ]]; then
    echo "Hit enter to continue with the release"
    read
else
    echo "Workspace is dirty, stash or commit before release!"
    exit 1
fi

./ci.sh
git push heroku master

git log -n 1 > .last-release
git commit -a -m "$(heroku releases -n 1)"
git push
```

It isn't by any means a replacement for a proper pipeline but it helped me feel
better about my releases.

Essentially all it is doing is ensuring that:
* You only release with a clean workspace. All changes you are running locally
  are deployed
* You can see what was last released by tracking the file
* A commit is only deployed if it passes the `ci.sh` script which for me usually
  contains running tests and linter
* A commit is also made containing the Heroku version that has been deployed

Hope you may find this useful. Any ideas or improvements let me know!

[Heroku]: https://www.heroku.com/
