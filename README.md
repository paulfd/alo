A LOoper

This is an LV2 plugin primarily targeted at the MOD Duo but hopefully it should
work on other systems too. It's based on the amp.c and metro.c lv2 example
plugins, plus some study of the loopor code.

The idea is to provide a mistake-proof way of creating and triggering live
music loops in sync with a click track. There's only one button, one loop.
Loop start/end is based on what's played, so the timing of the button hit
isn't critical.

The click/sync is assumed to be generated by something else (I'm using the
Simple Step Sequencer by x42): alo synchronises with midi beats/bars.


## design notes
```
              1       2       3       4       1       2
.-|-.-.-.-|-.-.-.-|-.-.-.-|-.-.-.-|-.-.-.-|-.-.-.-|-.-.-.   beats

<------always recording in the background--------------->

_______/<<^^^^^^^^^^^^^^^^^^^\____________________________  audio in
       |<<| phrase-start..goes here--> |<<|

      |^^^^^^^| hit loop button anytime in this region
                now we know loop starts at nearest beat 1 
                and threshold detect for intro phrase
          |^^^^^^^^^^^^^^^^^^^\_________<<|     loop is fixed length
                                       |  |     includes phrase-start
```

- we start in 'recording' mode

- when loop button is pressed:

  if 'recording' mode:
    - figure out when phrase-start happens
    - switch to loop_on mode at next phrase_start

  if loop_on mode:
    - play the loop from (and looping at) next phrase-start

  if loop_off mode:
    - stop playing loop at next phrase-start

  - if button is pressed twice within one beat, go back to 'recording' mode

## starting MOD docker build environment

docker run -ti --name mpb -p 9000:9000 -v ~/Projects/2018/moddevices/:/tmp/aloo-lv2 moddevices/mod-plugin-builder

## build notes
cd /home/builder/mod-plugin-builder
rm -fr  /home/builder/mod-plugin-builder/plugins/package/alo
cp -r /tmp/moddevices/alo /home/builder/mod-plugin-builder/plugins/package
rm -fr /home/builder/mod-workdir/plugins-dep/build/alo* && ./build alo

## deploy notes
cd /home/builder/mod-workdir/plugins && tar cz alo.lv2 | base64 | curl -F 'package=@-' http://192.168.51.1/sdk/install

## debug notes
tail -f /root/alo.log

mod-host -p 1234 -i
add http://devcurmudgeon.com/alo 0
