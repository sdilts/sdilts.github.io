---
layout: post
title: My Thoughts on Releasing My First Project on PyPI.
---

Last week I released my first piece of versioned software that is
published in a package repository.
The project itself, [cl-bindgen][1],
was received with mild excitement for a small amount of people.
This is exactly what I expected as it targets a very limited
audience. I'll write about the actual project more in the future,
but for now I'll settle with discussing how the actual release process
went, and what I would change in the future.

The actual mechanics of the release were fairly straightforward. The
first thing to do was tag the correct commit with a version number and
label it with the words "Initial Release". The next step was to package and
upload the source code to PyPI. There are numerous tutorials on how to
do this on the internet, and most of them have accurate and easy to
follow directions. However, out of all of the good tutorials out
there, the first one I picked had incorrect instructions for uploading
a project's README file to PyPI. After getting that sorted out, the
package was available to download from anywhere in the world by
running `pip install cl-bindgen` in a terminal. The last thing to do
was to post a release announcement on social media. [Here it is on
Reddit][2] if you want to see it.

Here's what went right with the process. Since I can really only gauge
how successful it was based on comments under the announcement and the
view count on Github, I can only draw limited conclusions to how it went:
+ The release announcement posts received some attention, and the post
  commenters ranged from mildly interested to excited. Considering how
  low traffic that subreddit is, I'm pleased with how well it was
  received.
+ Several of the commenters stated that it would be useful to them in
  the future, or wished that they'd had the tool in the past. Some of
  them had similar concerns that I did about the existing tooling, and
  were happy that a new tool addressing some of those concerns was
  created.
+ Quite a few people looked at the source code for the project, and
  I was pleasantly surprised how many of them kindly spoke up to fix
  minor problems.

Here's what went wrong:
+ All of those issues that people pointed out? 99% of them were
  misspellings or grammatical mistakes. I know I'm not the best
  proofreader, but there were a ton of them. I ended up releasing
  several patch versions (3 of them, to be exact) because I kept on
  finding minor spelling issues.
+ Although I tested the code thoroughly, I completely neglected to
  test for errors related to invalid C source code or libclang not
  being able to find header files. When all of these factors were
  correct, the program produced the correct output. When they weren't
  the program would happily carry on and produce the wrong output
  without any sign that things went wrong. These issues were fixed,
  but it was embarrassing to release the package with these errors present.
+ I released the code and set the version to `1.0.0` before I actually
  started using the software for myself. The version number is now at
  `1.1.0` because I added many quality of life improvements that
  should have been added before I did the official release.

Overall, I'm quite pleased with how the release went, but here are
things I'll do differently for the next release:
1. Run spellcheck on every file in the project, not just the README,
   and read it again for good measure. Running
   `ispell-comments-and-strings` within Emacs would have saved me a
   load of trouble.
2. Don't publish a project at version `1.0.0` as soon as you think it is minimally
   viable. There will be things you notice after using it
   for a while that will make you wish you had not set a major version
   number. Nothing in cl-bindgen is broken or awkward because I did
   this, but that doesn't mean it was a good idea.
3. Double check the API on any libraries you are using to see how they
   report errors. The Python bindings to Clang don't spit out errors
   like the C bindings do, and need to be manually checked.

For anyone who has every written a marginally useful piece of
open-source software, I encourage them to share it with others: my
experience was positive, and with any luck, yours should be
too.

[1]: https://github.com/sdilts/cl-bindgen
[2]: https://www.reddit.com/r/lisp/comments/f0cndg/announcing_clbindgen_a_command_line_tool_and/
