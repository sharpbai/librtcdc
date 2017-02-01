## Instructions

- Compile the C part. =cd ../src= and then make.
- Set the env variable: `export LD_LIBRARY_PATH=../src/vendor/build`
- Make a python2 `virtualenv` and `pip install cffi` there.
- Run `python ./setup.py install` to install the library. 
- Run `python ./dc_test.py` to run the tests.

## Using the python library

#### Example usage
```python
  from datachannel import DataChannel
  import time

  # Override the callbacks
  class Peer(DataChannel):
      def onOpen(self, channel):
          print "Data Channel opened"
      def onMessage(self, message):
          print "Message received: ", message
      def onClose(self, channel):
          print "Data Channel closed"
      def onChannel(self, peer, channel):
          print "Channel created"
      def onCandidate(self, peer, candidate):
          print "ICE Candidates found: ", candidate
      def onConnect(self, peer):
          print "Peer connection established"

  # Create two peers.
  peerA = Peer()
  peerB = Peer()

  offerSDP_A = peerA.generate_offer_sdp() # PeerA's "offer"
  candidateSDP_A = peerA.generate_local_candidate()
  candidateSDP_B = peerB.generate_local_candidate()

  offerSDP_B = peerB.parse_offer_sdp(offerSDP_A) # B is accepting A's offer and generates new offer SDP
  peerA.parse_offer_sdp(offerSDP_B) # This new offer SDP generated by B is to be parsed by A

  # Each peer should parse each other's candidate SDPs
  if (peerA.parse_candidates(candidateSDP_B) and peerB.parse_candidates(candidateSDP_A)):
      # Both peers would have established a peer connection and subsequently a data channel by now
      time.sleep(7)
      peerA.send_message("test") # Or peerB can do the same

```

#### Requirements

- Python 2
- CFFI >= v1.6