Forked from (https://github.com/ideo/LaughDetection), see readme there for details. I am unsatisfied by its custom training instructions, so here was what I did to make it work. I forget what I needed to pip install, but I don't think there are any new dependencies. Follow the instructions there until you can run `live_inference.py` and get real time measurements of laughter-confidence

I wanted real time feedback. In my use case, detecting female voices. 

# Custom Classes

I added `Notebooks/AudiosetProcessingGeneral` which enables you to specify the label you want, as the main repo hard coded the word 'laughing' everywhere. You should only have to modify the variables at the top of the file.

The ontology [here](https://research.google.com/audioset/ontology/index.html) will give you the label data below. If you want all the labels in one file instead of organized hierarchically, [this](https://github.com/IBM/audioset-classification/blob/master/audioset_classify/metadata/class_labels_indices.csv) will give you everything you need to simply get going. 

For my use case, I made the following files:

`Data/female_labels.csv `
```
2,/m/02zsn,"Female speech, woman speaking"
33,/t/dd00004,"Female singing"
```

`Data/not_female_labels.csv `
```
1,/m/05zppz,"Male speech, man speaking"
32,/t/dd00003,"Male singing"
```
The "not_" prefix is to stay in keeping with the original repo's naming convention. Since its use case was laughter vs speech it made more sense to keep it that way for the general case. 

For your class, change "female" in the filenames to whatever you want to detect, and change the contents of the file to the appropriate labels from the ontology mentioned before. Keep in mind the context you're detecting it from. As an example, the laughter detector tried to distinguish laughter from speech. 

now run `jupyter-notebook`, open up `Notebooks/AudiosetProcessingGeneral`. In it you will see a line `pos = "female"`. Change 'female' to the same thing as your filenames. 

You should be able to just run the notebook now, although I had trouble with two things:

I had to run `wget http://storage.googleapis.com/us_audioset/youtube_corpus/v1/features/features.tar.gz` manually. This is a large file, so it will take a while to download.

My positive class had fewer examples than my negative class, which gave an error. So if you get an error in this step, switch the sampling to the other set of labels.
```
negative = labels[labels[neg]==True]#.sample(positive.shape[0])
positive = labels[labels[pos]==True].sample(negative.shape[0])
```

The train set took me an hour to generate. 

# Training

You can use `Notebooks/ModelTraining.ipynb` normally, but you'll have to update `post = "female"` here as well. Make sure it saves, and keep track of the model you trained. If you trained `LSTM_SingleLayer_100Epochs.h5` then it will work just fine in testing. If not, see next section. 

# Testing
I found pavucontrol's [monitors](https://unix.stackexchange.com/questions/82259/how-to-pipe-audio-output-to-mic-input/82297) the most reliably way to test the setup. My own microphone is sadly very low quality, but I could simply direct a youtube video at this and see consistent results.

If you picked a different model you'll need to change `'keras_model', 'Models/LSTM_SingleLayer_100Epochs.h5',` in live_plotting.py to the model you chose.

Run `python live_plotting.py`, and it should just work. 

You can play around with `sample_length` and `RATE` if you want.

[This](https://www.youtube.com/watch?v=21ZfGPp-Ves) is a wonderful video to test it with. 

I've found that LSTM_SingleLayer_100Epochs.h5 at sample_length 3.0 and RATE 16000 works perfectly with the above video. Shockingly reliably. 
