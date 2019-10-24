python ECG API
============

**Overview:**

Connects to node server, crunches and returns incoming ECG data over ZeroMQ (websockets). Has code setup for Postgres connection if desired.

The Python server is set up to recieve an object that, once parsed by JSON, follows the following format:
{'amplitude': <FLOAT @ 3 decimal places>, 'time': <ISO 8601 Timestamp>}

**Details:**

The python server is instantiated using ZeroRPC and the server's RPC class exposes the methods hello() and cruch() to Node. A new RWaveAnalyzer is instantiated with user-defined parameters.

The R-Wave analyzer is currently bare-bones and calculates BPM and checks for signs of arrhythmia. Many more features can be built in. Below is a rundown of user-defined parameters that are passed into RWaveAnalyzer:

rWaveNotch: 
A measure of amplitude over which the Analyzer will start recording amplitudes to extract the r-wave peak.

thresholdForFeatureCount: 
The total number of abnormal RR-interval features found within the RWaveBuffer that would indicate an issue like arrhythmia.

timeSpan: 
Minimum duration of RR-interval that constitutes normal rythm, anything below indicates abnormal interval.

bufferLength: 
The number of R-waves to store and check against thresholdForFeatureCount

After RWaveAnalyzer is initialized, the following processes occur as data comes in:

Crunch() is called from within the node server and passed packets of ECG data. The crunch() method converts the incoming data into a JSON object and passes that object to the analyze method of the RWaveAnalyzer.

Inside of RWaveAnalyzer, the JSON object is sent through analyze() into the findRWavePeak() method where R wave peak times are found and extracted. Those R wave peak times are added into an r-wave buffer. The analyze() function always returns the result of the call to checkBuffer(), which calculates the distances between r-wave peaks, checks those time distances for abnormalities, and also passes those time distances to the getHeartRate method to calculate BPM. If there are enough abnormalities to break the featureThreshold, the checkBuffer() method returns a '404' status code along with the heartRate. Otherwise, it will return a '202', or NSR, code along with the heartRate. The statusCode/heartRate object is passed back to Node as the return value of crunch().

The testData file can be used to test run a real dataset (included) through the RWaveAnalysis script.
