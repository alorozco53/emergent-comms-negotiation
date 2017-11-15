# "Emergent Communication through Negotiation"

Reproduce https://openreview.net/forum?id=Hk6WhagRW&amp;noteId=Hk6WhagRW , "Emergent Communication through Negotation", ICLR 2018 anonymous submission.

## To install

- install pytorch 0.2, https://pytorch.org
- download this repo, `git clone https://github.com/asappinc/emergent_comms_negotiation`

## To run

```
python ecn.py [--disable-comms] [--disable-proposal] [--disable-prosocial] [--enable-cuda] [--term-entropy-reg 0.5] [--utterance-entropy-reg 0.0001] [--proposal-entropy-reg 0.01] [--model-file model_saves/mymodel.dat] [--name gpu3box]
```

Where options are:
- `--enable-cuda`: use NVIDIA GPU, instead of CPU
- `--disable-comms`: disable the comms channel
- `--disable-proposal`: disable the proposal channel (ie agent can create proposals, but other agent cant see them)
- `--disable-prosocial`: disable prosocial reward
- `--term-entropy-reg VALUE`: termination policy entropy regularization
- `--utterance-entorpy-reg VALUE`: utterance policy entropy regularization
- `--proposal-entropy-reg VALUE`: proposal policy entropy regularization
- `--model-file models_saves/FILENAME`: where to save the model to, and where to look for it on startup
- `--name NAME`: this is used in the logfile name, just to make it easier to find/distinguish logfiles, no other purpose

## Stdout layout

eg if we have:
```
   000000 4:4/0 7:5/5 9:4/4
                                      000000 4:5/0 6:1/5 7:2/4
   000000 4:0/0 7:0/5 9:1/4
                                      ACC
  r: 0.91
```

Then:
- each of the first 4 lines here is the action of a single agent
- the `ACC` line is the agent accepting previous proposal
- each proposal line is laid out as:
```
  [utterance]   [utility 0]:[proposal 0]/[pool 0] ... etc ...
```
- if the agents run out of time, last line will be `[out of time]`

One negotation is printed out every 3 seconds or so, using the training set; the other negotations are executed silently.  There is no test set for now.

## Results so far

### proposal, comms, prosocial

Graphs for three training runs, with identical settings:

<img src="images/comms_prop_soc_tests_threerunsb.png?raw=true" width="800" />

This is using the following entropy regularizations:
- termination: 0.5
- utterance: 0.0001
- proposal: 0.01

This seems approximately in line with the results in Table 1 of the paper. This would be 'Prosocial, random termination, Both: Fraction of joint reward'

Note that the difference between train and test:
- train are essentially on randomly drawn games, albeit disjoint with test games. Stochastic draws are enabled
- test is on a fixed set of 5 batches of 128 games. Stochasticity is disabled, and the greedy/argmax action is taken

### proposal, no comms, prosocial

<img src="images/20171115_prop_nocomms_soc.png?raw=true" width="800" />

### no proposal, comms, prosocial

<img src="images/20171115_noprop_comms_soc.png?raw=true" width="800" />

### no proposal, no comms, prosocial

<img src="images/20171115_noprop_nocomms_soc.png?raw=true" width="800" />

### proposal, comms, no social

<img src="images/20171115_prop_comms_nosoc.png?raw=true" width="800" />

## Unit tests

- install pytest, ie `conda install -y pytest`, and then:
```
py.test -svx
```
- there are also some additional tests in:
```
python net_tests.py
```
(which allow close examination of specific parts of the network, policies, and so on; but which arent really 'unit-tests' as such, since neither termination criteria, nor success criteria)

## Plotting graphs

__Assumptions__:
- running the training on remote Ubuntu 16.04 instances
  - `ssh` access, as user `ubuntu`, to these instances
  - remote has home directory `/home/ubuntu`
  - logs are stored in subdirectory `logs` of current local directory
  - the location of `logs` relative to `~` is identical on local computer and remote computer

__Setup/configuration__:
- copy `instances.yaml.templ` to `~/instances.yaml`, on your own machine
  - configure `~/instances.yaml` with:
    - name and ip of each instance (names are arbitrary)
    - the path to your private sshkey, that can access these instances

__Procedure__
- run:
```
python merge.py --hostname [name in instances.yaml] [--logfile logs/log_20171104_1234.log] \
    [--title 'my graph title'] [--y-min 75 --y-max 85]
```

This will:
- `rsync` the logs from the remote instance identified by `--hostname`
- if `--logfile` is specified, load the results from that logfile
  - else, will look for the most recent logfile, ordered by name
- plots the graph into `/tmp/out-reward.png`
