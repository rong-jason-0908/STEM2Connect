STEM2Connect Hackathon Devlog - Jason Rong
The Tremor Detector 3000

Overview:
It’s a tool that detects Parkinson’s-type tremors (4-6Hz) from a spiral drawn by a user with their mouse.

General info:
	Spiral drawings are a classic bedside test to detect tremors
	This test, as of right now, is based on subjective judgment and requires the patient to schedule an actual clinician for a simple test
	Parkinson’s is often caught very late in its development process, since physical symptoms appear only after a large portion of dopamine-producing cells have died
	An early 5Hz tremor caught by the model is almost invisible to the naked eye, but could warn someone to go to their doctor and get tested more formally
	Issues:
		Can’t actually diagnose, only screen
		No patient history access for better information
		Online data is virtually non-existent (though we somewhat bypass this)
Data synthesis:
	UCI 395: tablet pen trajectories, 61 Parkinson’s patients and 15 controls
	Public mouse datasets were unusable
		31-person gamified set was too small
		261 person set recorded the cursor only every 500 ms, so it can’t properly detect a 4-6 Hz tremor
	Input is a time series of (x,y,t), but it is reduced to 8 features, so training compute is tiny
Architecture V1
	Resample to 100 Hz, then normalize by the spiral’s size
	Savitzky-Golay low-pass keeps the smooth spiral; subtracting it leaves the wobble (the tremor) for the model
	Welch power spectrum of the wobble to average and smooth results instead of a single Fourier transform (too noisy)
	V1 gave a balanced accuracy of 0.839 (AUROC 0.936) on real tablet cases in cross-validation; a model trained only on simulated data reached an AUROC of 0.906 on real cases
	Issues:
		Adding simulated data to training did not significantly improve accuracy (synthetic data wasn’t good enough to mimic the tablet data)
		Pressure and total duration are specific to tablet instead of mouse, so we left them out of the features

Issues with V1:
	The tablet trained model failed on the mouse inputs
	Collected 16 data points from healthy mouse users (healthy friends and family); only 19% were correctly cleared (3 of 16)
	Only healthy volunteers available, so no way to see how it does on mouse input from people WITH Parkinson’s (though it flagged most healthy people, so the model was unusable anyway)
V2 and V3 development:
	V2: baseline alignment, implemented a new metric z = (feature - device-healthy median) / device-healthy IQR so that our z score is adjusted for mouse shakiness
	V3: Adds injected tremor copies of real mouse traces. Ended with 94% of healthy mouse users correctly cleared (15 of 16)
	Final specificity on healthy mouse users: 94% with injected-tremor training set
	Issues:
		Tablet accuracy dropped slightly
		Still got 1/16 wrong
		Risk of overfitting to injected tremor (checked by testing injected tremor at 2 to 12 Hz)
		Alignment depends on a base of just 16 people
Creating browser interface
	Scoring is moved to Javascript
	“How score is built” panel shows all 8 metrics used
	Replay with simulated tremor adds 5Hz to the entire drawing to show what a standard tremor might look like
	Demo has to be an HTML, localhost had some issues
Trackpads are awful
	3 healthy trackpad drawings scored 0.8, 0.43, and 0.51 against a mouse baseline
	The trackpad is jerky, and adds to the tremor reading
	So we made it mouse only
