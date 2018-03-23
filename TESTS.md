# Test of the Dezyne model

We have applied both Verification and Validation.

## Verification

Verification of the model, in particular the ConsensusModule and its interfaces, was performed via the Dezyne Model Verification facility in the Dezyne IDE.
This facility is based on the [mCRL2 modelchecker](http://www.mcrl2.org/).

The actual verification log of the ConsensusModule:

	dzn$ dzn --session=49 -v verify --model=ConsensusModule -I raft-dezyne-model raft-dezyne-model\ConsensusModule.dzn --version=2.7.0
	working directory: D:\OneDrive\Documents\GIT\GitHub\dortmans
	......................................................
	verify: IServer: check: deadlock: ok                   
	....
	verify: IServer: check: livelock: ok                   
	...
	verify: ITimer: check: deadlock: ok                   
	....
	verify: ITimer: check: livelock: ok                   
	......
	verify: IRules: check: deadlock: ok                   
	.....
	verify: IRules: check: livelock: ok                   
	...
	verify: IConfig: check: deadlock: ok                   
	....
	verify: IConfig: check: livelock: ok                   
	...
	verify: IRpcClient: check: deadlock: ok                   
	....
	verify: IRpcClient: check: livelock: ok                   
	...
	verify: IRpcServer: check: deadlock: ok                   
	....
	verify: IRpcServer: check: livelock: ok                   
	.....
	verify: ConsensusModule: check: deterministic: ok                   
	.......
	verify: ConsensusModule: check: illegal: ok                   
	........
	verify: ConsensusModule: check: deadlock: ok                   
	........
	verify: ConsensusModule: check: livelock: ok                   
	...........
	verify: ConsensusModule: check: compliance: ok  

The latest version of our model has no verification errors as can be seen in above verification result log..
	
## Validation

We have also used the Dezyne Model Simulation service to validate the states and state transitions as described in the Raft protocol.

Typical scenarios can be executed in the Dezyne IDE by starting Dezyne Model Simulation on the ConsensusModule and pressing the events buttons that we have specified below.

Bring Server online:

	ctrl.OnLine
	rules.return

Leader election after election timeout:
	
	electionTimer.timeout
	rpcclient.RequestVoteGranted
	rpcclient.RequestVoteGranted
	rules.leadershipWon
	
Leader handles new request from Client, replicates it and replies the result

	rpcserver.ClientRequest

Leader replicates its logged client commands every heartbeat

	heartbeatTimer.timeout
	heartbeatTimer.timeout
	...
	
Leader executes command after replication and replies the result to client 
	rules.ClientRequestSucess

Leader looses leadership and becomes Follower:

	heartbeatTimer.timeout
	rpcclient.AppendEntriesRefused
	rules.leadershipLost
