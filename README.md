
## Generic session manager for Emacs

This is a brief overview. Please see the code for more details.

Sesman provides facilities for session management and interactive session association with current contexts. While "sessions" is a broad and implementation specific concept, the primary target of `sesman` are Emacs based IDEs ([CIDER][], [ESS][], [Geiser][], [Robe][], [SLIME][] etc.)

For Emacs based IDEs, session is commonly composed of one or more physical processes (sub-processes, sockets, websockets etc). For example in the current implementation of [CIDER][] a session would be composed of one or more cider connections (Clojre or ClojureScript). Each [CIDER][] connection consists of user REPL buffer and two sub-processes, one for user eval communication and another for tooling (completion, inspector etc).

### Concepts:

  - "session" is a list of the form `(session-name ..other-stuff..)` where `..other-stuff..` is system dependent.
  - "system" is generic name used for a tool which uses sesman (e.g. `CIDER`, `ESS` etc)
  - "contexts" are Emacs objects which describe current context. For example `current-buffer`, `default-directory` and `project-current` are such contexts. Context objects are used to create associations (links) between the current context and sessions. At any given time the user can link (unlink) sessions to (from) contexts. By default there are three types of contexts - buffer, directory and project, but systems can define their own contexts.
  
Sesman is composed of two parts, [user interface][], available as [sesman map][], and [system interface][] consisting of a few generic functions for which each systems should define its own methods. 

### [User Interface][]

Consists of 

 - lifecycle management commands (`sesman-start`, `sesman-kill` and `sesman-restart`), and
 - association management commands (`sesman-link-with-buffer`, `sesman-link-with-directory`, `sesman-link-with-project` and `sesman-unlink`). 

From the user's prospective the work-flow is simple. Start a session, either with `sesman-start` (`C-c C-s C-s`) or some of the system specific commands (`run-xyz`, `xyz-jack-in` etc). On startup each session is automatically associated with the least specific context (commonly a project). In the most common case the user has only one session open per project. In such case, no ambiguity arises when a system retrieves the current session. If none or multiple sessions are associated with current context, the user is asked to resolve the ambiguity. Currently, in order to resolve ambiguity in presence of multiple sessions per project the user has to associates the session with a more specific contexts (directory or buffer). In the future there will be a provision to resolve the ambiguity automatically through the recency ordering (see below).

Currently there is only one custom variable, `sesman-1-to-1-links`, which lists context types for which `1-to-1` associations are desired (defaults to `'(directory buffer)`. This means, that each time the user links a session with a directory, any previous associations with that directory are lost. For context types not in this list (e.g. `project`), 1-to-many associations are allowed. 

### [System Interface][]

Consists of several generics, of which only first two are strictly required:

  - `sesman-start-session`
  - `sesman-kill-session`
  - `sesman-restart-session` - defaults to `sesman-start-session` + `sesman-kill-session`
  - `sesman-more-relevant-p` - used for sorting sessions in "recency" order. Defaults to sorting by session name.
  <!-- - `sesman-friendly-session-p` - used to define friendly sessions (e.g. dependency projects). -->
  
Depending on the purpose at hand, sesman system can use several functions to retrieve sessions (`sesman-ensure-linked-session`, `sesman-linked-sessions`, `sesman-friendly-sessions`  and `sesman-sessions`). Most important of these being `sesman-ensure-linked-session` which should be used to ensure that at least one session is linked to the current context. It returns the most specific session given sesman associations already in place. In case of ambiguity (or no sessions) the user is asked for a session.

Systems could directly use user level commands to manage sessions (`sesman-start`, `sesman-kill`) or use legacy system specific initializer (`run-xyz`, `xyz-jack-in` etc). In the latter case, systems should call `sesman-register` to register their sessions with `sesman`.

Systems should link [semsna map][] into their modes' key-maps (ideally on `C-c C-s`, which is a good mnemonic, is free in CIDER and already does similar things in ESS).


[user interface]: https://github.com/vspinu/sesman/blob/master/sesman.el#L53
[system interface]: https://github.com/vspinu/sesman/blob/master/sesman.el#L133
[sesman map]: https://github.com/vspinu/sesman/blob/master/sesman.el#L112-L130

[cider]: https://github.com/clojure-emacs/cider
[ess]: https://ess.r-project.org/
[geiser]: https://github.com/jaor/geiser
[robe]: https://github.com/dgutov/robe
[slime]: https://common-lisp.net/project/slime/
