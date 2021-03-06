ex1_relup
=========

An OTP release demonstrating release upgrade technique.
There is no production use, it's only for a lesson on live upgrades.

[Lesson content](LESSON1.md).

Exercise 1. OTP Upgrades
------------------------

Steps to create project, version 0.1.0, git tag 0.1.0

    Start umbrella project:
      $ rebar3 new release ex1_relup
    ...add new gen_server ex1_relup.erl, add handle_info & timer start
    ...add childspec to supervisor: #{id => ex1_relup
    ...add variable to ex1_relup.app.src, env: {ticker, 1000}
    <(instead of all steps above simply 'git checkout 0.1.0')>
    Create a deployment package:
      $ rebar3 as prod release
      $ rebar3 as prod tar
    Deploy to your server:  
      mkdir ~/server
      mv _build/prod/rel/ex1_relup/ex1_relup-0.1.0.tar.gz ~/server
      cd ~/server
      tar xvzf ex1_relup-0.1.0.tar.gz
       bin/ex1_relup console
    ...keep it running ... and get back to coding ...

Version 0.2.0 with corresponding git tag

    ...change env to {ticker, 5000}, app version to 0.2.0, release to 0.2.0
    <(or just git checkout 0.2.0)>
    Create release 0.2.0
       $ rebar3 as prod release
    ...create ex1_relup.appup, either manually or 
    use rebar3 appup generate: add plugin to rebar.config)
      $ rebar3 as prod appup generate
    Create relup script for hot release upgrade  
      $ rebar3 as prod relup -n ex1_relup -v "0.2.0" -u "0.1.0"
    Create deployment package  
      $ rebar3 as prod tar
    Deploy code to server
      $ mkdir ~/server/releases/0.2.0  
      $ mv _build/prod/rel/ex1_relup/ex1_relup-0.2.0.tar.gz ~/server/releases/0.2.0/
      $ cd ~/server
    Perform hot code upgrade:  
      $ ./bin/ex1_relup upgrade "0.2.0"
       Release 0.2.0 not found, attempting to unpack releases/0.2.0/ex1_relup-0.2.0.tar.gz
       Unpacked successfully: "0.2.0"
       Installed Release: 0.2.0
       Made release permanent: "0.2.0"

Downgrade back to 0.1.0:

        $ ./bin/ex1_relup downgrade "0.1.0"
        Release 0.1.0 is marked old, switching to it.
        Installed Release: 0.1.0
        Made release permanent: "0.1.0"

Version 0.3.0, tag 0.3.0

    ...remove {ticker, 5000} from application env
    hardcode 500 in ex1_relup.erl
    change versions in ex1_relup.app.src and rebar.cofig
    <(or just git checkout 0.3.0)>
      $ rebar3 as prod release
      $ rebar3 as prod appup generate --previous_version "0.2.0" % generate ex1_relup.appup file
      $ rebar3 as prod relup -n ex1_relup -v "0.3.0" -u "0.2.0"
      $ rebar3 as prod tar
      $ mkdir ~/server/releases/0.3.0
      $ mv _build/prod/rel/ex1_relup/ex1_relup-0.3.0.tar.gz ~/server/releases/0.3.0/
      $ cd ~/server/
      $ ./bin/ex1_relup upgrade "0.3.0"
    ...experience failure, as we downgraded to 0.1.0, but made appup from 0.2.0 only
      $ ./bin/ex1_relup upgrade "0.2.0"
      $ ./bin/ex1_relup upgrade "0.3.0"

Break things a bit, as application controller table is public:

    > ets:lookup(ac_tab, {env, ex1_relup, ticker}).

Without all the releases

    > ets:insert(ac_tab, {{env, ex1_relup, ticker}, 3000}).
    
More to appup

    More than code:load():
    * start or stop servers
    * reinitialise supervisors
    * perform generic apply(M, F, A)
    * unload old modules
    * restart entire VM



Exercise 2: Configuration Delivery Pipeline
-------------------------------------

Implementing separate pipeline is complicated, step-by-step guide is
not as simple and clear as for first exercise.

    <(git checkout 0.4.0)>
      echo "{ticker, 2000}." > /tmp/cdp
    Let's use rebar3 shell for simplicity
      $ rebar3 shell
    attempt configuration change:
      echo "broken" >> /tmp/cdp.new
    look into /tmp/cdp.rej to see rejected change  
    make configuration change  
      echo "ex1_relup:ex1_relup:ticker:1500" >> /tmp/cdp.new
      
