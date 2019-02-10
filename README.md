# Sourcegraph Local

A quick way to run a [Sourcegraph][sg] server that points to local git
repositories.

[sg]: https://sourcegraph.com

## Usage

0. Given that your local git repositories all live under a path like:

   ```
    ${HOME}/dev/

    # Under dev/ we have individual git repositories
    ${HOME}/dev/git-example-1
    ${HOME}/dev/git-example-2
    ...
   ```

1. Then just run with [docker-compose][dc] like:

   ```
   SOURCEGRAPH_LOCAL_REPOS_ROOT=${HOME}/dev/ docker-compose up
   ```

   Note that we specify the environment variable `SOURCEGRAPH_LOCAL_REPOS_ROOT`
   to point to our local git repository root path.

2. Next, visit http://localhost:7080 in your browser to load the Sourcegraph
   web UI.

3. (For first time setup only) Finally, in Sourcegraph's Site admin page
   (http://localhost:7080/site-admin/external-services/add):

   - Set a *Display name* (e.g., `local`).
   - Set *Kind* to `Other`.
   - Then in the configuration box:
     
     ```
     {
       // Supported URL schemes are: http, https, git and ssh
       "url": "git://git/",

       // Repository clone paths may be relative to the url (preferred) or absolute.
       "repos": [
         "git-example-1",
         "git-example-2"
       ]
     }
     ```

     Note that the `url` field points to the locally running git daemon (which
     is serving your local git repositories as read-only). Each repository must
     be manually added to the `repos` list.

[dc]: https://docs.docker.com/compose/

## How does this work?

Sourcegraph does have a way to [add repositories to local disk][sg-local-disk].
However, it requires cloning your existing repositories and the process is
a bit tedious. They are [planning to make this easier][sg-easier-add].

But if there were already a `git`/`http(s)`/`ssh` endpoint that serves git
repositories, then it is fairly easy to point Sourcegraph to this endpoint and
have it automatically mirror these repositories. That's the general idea so
"Sourcegraph Local" is just a simple `docker-compose.yml` script that starts
a [git daemon][git-daemon] server that is accessible to the Sourcegraph server.

[sg-local-disk]: https://docs.sourcegraph.com/admin/repo/add_from_local_disk
[sg-easier-add]: https://github.com/sourcegraph/sourcegraph/issues/1527
[git-daemon]: https://git-scm.com/docs/git-daemon

## Even simpler approach

If this approach is too complicated, there is even a simpler method:

1. On your local machine, `cd` to your git repositories root and manually start
   the git daemon:

   ```
   git daemon --reuseaddr --base-path=. --export-all --verbose
   ```

2. Start a Sourcegraph server (e.g., using their
   [Quickstart guide][sg-quickstart]).

3. Configure the Sourcegraph server's external service to use the url:
   
   ```
   {
     // Supported URL schemes are: http, https, git and ssh
     "url": "git://host.docker.internal/",

     // Repository clone paths may be relative to the url (preferred) or absolute.
     "repos": [
       "git-example-1",
       "git-example-2"
     ]
   }
   ```

   Where `host.docker.internal` points to your host machine's address. The
   downside of this approach is that it only works for Mac and Windows docker
   servers ([Linux docker lacks this helper hostname][hdi-linux]).

[sg-quickstart]: https://docs.sourcegraph.com/#quickstart-guide
[hdi-linux]: https://stackoverflow.com/a/50057593/66771