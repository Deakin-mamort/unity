# Unity Timeline

Timeline is Unity's solution to an in-built video editor (e.g. Premiere Pro, After Effects and Final Cut Pro), out of the box unity timeline provides the following track types

 - Play animations clips
 - Play audio files
 - Turn objects on\off

# Timeline Custom Scripts

Unity Timeline allows developers to create their own custom scripts for timeline tracks and clips . 

## Track Script

Track script describes track data
- ```TrackClipType``` requires ```PlayableAsset```
- ```TrackBindingType``` requires ```GameObject```, ```Component``` or ```Asset``` 

```csharp
//Track.cs

[TrackClipType(typeof(ClipAsset))]
[TrackBindingType(typeof(BindingType))]
public class Track : TrackAsset {}
```

## Clip Scripts

A clip contains data in a ```PlayableAsset``` with behaviour described in ```PlayableBehavior```. 

```csharp
//ClipAsset.cs

public class ClipAsset : PlayableAsset
{
   // Required properties needed to be changed during playback
   public bool foo = true;
   public float bar = 1f;
   
   //playable asset that can be loaded into track
   public override Playable CreatePlayable (PlayableGraph graph, GameObject owner)
   {
       var playable = ScriptPlayable<ClipBehaviour>.Create(graph);
      
       var Behaviour = playable.GetBehaviour();
       Behaviour.foo = foo;
       Behaviour.bar = bar;

       return playable;
   }
}
```

A ```PlayableBehaviour``` holds the logic that will be executed on each frame to describe how the clip will play  

```csharp
//ClipBehaviour.cs

public class ClipBehaviour : PlayableBehaviour
{
   public bool foo = true;
   public float bar = 1f;

   public override void ProcessFrame(Playable playable, FrameData info, object playerData)
   {
       BindingType object = playerData as BindingType;

       if (object != null)
       {
           //Behaviour updated each frame
       }
   }
}
```

## Mixer Script

Mixer script describes the transition between two or more tracks, the track script must ```CreateTrackMixer```

```csharp
// Track.cs

[TrackClipType(typeof(ClipAsset))]
[TrackBindingType(typeof(BindingType))]

public class Track : TrackAsset
{
    public override Playable CreateTrackMixer(PlayableGraph graph, GameObject go, int inputCount) {
        return ScriptPlayable<LightControlMixerBehaviour>.Create(graph, inputCount);
    }
}
```
When we want a track to mix we need to move the behaviour to mixer script 
```csharp
// LightControlBehaviour.cs

public class LightControlBehaviour : PlayableBehaviour
{
    public bool foo = true;
    public float bar = 1f;
    //Move behaviour to MixerBehaviour class
}
```
Mixer script describes play behaviour taking into consideration the weight values of each clip on the track for transition effects.
```csharp
// LightControlMixerBehaviour.cs

public class MixerBehaviour : PlayableBehaviour
{
    // NOTE: This function is called at runtime and edit time.  Keep that in mind when setting the values of properties.
    public override void ProcessFrame(Playable playable, FrameData info, object playerData)
    {
        BindingType trackBinding = playerData as BindingType;
        float mixedBar = 1f;

        if (!trackBinding)
            return;

		//number of clips on track
        int inputCount = playable.GetInputCount (); 
		
		for (int i = 0; i < inputCount; i++)
        {
            float inputWeight = playable.GetInputWeight(i);
            ScriptPlayable<ClipBehaviour> inputPlayable =(ScriptPlayable<ClipBehaviour>)playable.GetInput(i);
            ClipBehaviour input = inputPlayable.GetBehaviour();
            
            //mix results for transition
            mixedBar += input.Bar * inputWeight;
        }

        //final transition values
        trackBinding.bar= mixedBar;
    }
}
```


