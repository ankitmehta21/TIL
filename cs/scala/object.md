In scala, an object is a special class which can only have one instance.
As far as I can tell, objects are meant to provide class level functions instead of instance level functions.
In telemetry.views it looks like we're using objects to organize our code for each view. 
It doesn't make sense in this context to have multiple instances of a LongitudinalView, so the singleton class is a nice representation.
