STEM2Connect Hackathon Devlog - Jason Rong<br>
The Tremor Detector 3000<br>

Overview:<br>
It’s a tool that detects Parkinson’s-type tremors (4-6Hz) from a spiral drawn by a user with their mouse.<br>

General info:<br>
&emsp;&emsp;Spiral drawings are a classic bedside test to detect tremors<br>
&emsp;&emsp;This test, as of right now, is based on subjective judgment and requires the patient to schedule an actual clinician for a simple test<br>
&emsp;&emsp;Parkinson’s is often caught very late in its development process, since physical symptoms appear only after a large portion of dopamine-producing cells have died<br>
&emsp;&emsp;An early 5Hz tremor caught by the model is almost invisible to the naked eye, but could warn someone to go to their doctor and get tested more formally<br>
&emsp;&emsp;Issues:<br>
&emsp;&emsp;&emsp;&emsp;Can’t actually diagnose, only screen<br>
&emsp;&emsp;&emsp;&emsp;No patient history access for better information<br>
&emsp;&emsp;&emsp;&emsp;Online data is virtually non-existent (though we somewhat bypass this)<br>
Data synthesis:<br>
&emsp;&emsp;UCI 395: tablet pen trajectories, 61 Parkinson’s patients and 15 controls<br>
&emsp;&emsp;Public mouse datasets were unusable<br>
&emsp;&emsp;&emsp;&emsp;31-person gamified set was too small<br>
&emsp;&emsp;&emsp;&emsp;261 person set recorded the cursor only every 500 ms, so it can’t properly detect a 4-6 Hz tremor<br>
&emsp;&emsp;Input is a time series of (x,y,t), but it is reduced to 8 features, so training compute is tiny<br>
Architecture V1<br>
&emsp;&emsp;Resample to 100 Hz, then normalize by the spiral’s size<br>
&emsp;&emsp;Savitzky-Golay low-pass keeps the smooth spiral; subtracting it leaves the wobble (the tremor) for the model<br>
&emsp;&emsp;Welch power spectrum of the wobble to average and smooth results instead of a single Fourier transform (too noisy)<br>
&emsp;&emsp;V1 gave a balanced accuracy of 0.839 (AUROC 0.936) on real tablet cases in cross-validation; a model trained only on simulated data reached an AUROC of 0.906 on real cases<br>
&emsp;&emsp;Issues:<br>
&emsp;&emsp;&emsp;&emsp;Adding simulated data to training did not significantly improve accuracy (synthetic data wasn’t good enough to mimic the tablet data)<br>
&emsp;&emsp;&emsp;&emsp;Pressure and total duration are specific to tablet instead of mouse, so we left them out of the features<br>

Issues with V1:<br>
&emsp;&emsp;The tablet trained model failed on the mouse inputs<br>
&emsp;&emsp;Collected 16 data points from healthy mouse users (healthy friends and family); only 19% were correctly cleared (3 of 16)<br>
&emsp;&emsp;Only healthy volunteers available, so no way to see how it does on mouse input from people WITH Parkinson’s (though it flagged most healthy people, so the model was unusable anyway)<br>
V2 and V3 development:<br>
&emsp;&emsp;V2: baseline alignment, implemented a new metric z = (feature - device-healthy median) / device-healthy IQR so that our z score is adjusted for mouse shakiness<br>
&emsp;&emsp;V3: Adds injected tremor copies of real mouse traces. Ended with 94% of healthy mouse users correctly cleared (15 of 16)<br>
&emsp;&emsp;Final specificity on healthy mouse users: 94% with injected-tremor training set<br>
&emsp;&emsp;Issues:<br>
&emsp;&emsp;&emsp;&emsp;Tablet accuracy dropped slightly<br>
&emsp;&emsp;&emsp;&emsp;Still got 1/16 wrong<br>
&emsp;&emsp;&emsp;&emsp;Risk of overfitting to injected tremor (checked by testing injected tremor at 2 to 12 Hz)<br>
&emsp;&emsp;&emsp;&emsp;Alignment depends on a base of just 16 people<br>
Creating browser interface<br>
&emsp;&emsp;Scoring is moved to Javascript<br>
&emsp;&emsp;“How score is built” panel shows all 8 metrics used<br>
&emsp;&emsp;Replay with simulated tremor adds 5Hz to the entire drawing to show what a standard tremor might look like<br>
&emsp;&emsp;Demo has to be an HTML, localhost had some issues<br>
Trackpads are awful<br>
&emsp;&emsp;3 healthy trackpad drawings scored 0.8, 0.43, and 0.51 against a mouse baseline<br>
&emsp;&emsp;The trackpad is jerky, and adds to the tremor reading<br>
&emsp;&emsp;So we made it mouse only<br>

