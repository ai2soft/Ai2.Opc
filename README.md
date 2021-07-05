# Ai2.Opc

This is a library using for read / write opc tag values . Using OpcController to connect to a KepServer , setup an OPCGroup to read all user defined tags.

# How to use

  1. Reference the necessary dll  "Ai2.Opc.dll" to your project , and add "Interop.OPCAutomation" to your project bin folder.

  2. Add a sub-class "xxxOpcController", which inherits from "OpcController" , into you project.

     for example :

```c#
 class TruckSampleOpcController : OpcController
    {
        public TruckSampleOpcController(IOpcLogger logger, bool isReadOnly, CancellationToken token) : base(logger, isReadOnly, token)
        {
            Register<Tags>();
        }

        public void ConnectToOpcServer()
        {
            var svr = new OpcServer { ServiceName = "KEPware.KEPServerEx.V4" };
            Connect(svr);
        }

        /// <summary>
        /// register callbacks
        /// </summary>
        public override void RegisterActions()
        {
            //RegisterMultiConditionAsyncCallBack(
            //    OnAsyncSysRunning.BuildAction()
            //        .DependenceOn(Tags.SystemRunning, true)
            //);

        }


        public void LeftMove()
        {
            Tags.LeftMove.WriteSwitch();
        }

        public void RightMove()
        {
            Tags.RightMove.WriteSwitch();
        }

        public void ChangePoint()
        {
            Tags.ChangePoint.WriteSwitch();
        }

        public void Start()
        {
            //todo ....

            Tags.StartSys.WriteSwitch();
        }

        public void Stop()
        {
            Tags.StopSys.WriteSwitch();
        }

        public void ReturnToOriginalPlace()
        {
            Tags.ReturnToOriginalPlace.WriteSwitch(isOnlyOn:true);
        }

        public void ReturnToTokenPlace()
        {
            Tags.ReturnToTokenPlace.WriteSwitch();
        }

        public void CloseHopper()
        {
            Tags.CloseHopper.WriteSwitch();
        }
    }
```

 3. What does "RegisterActions()" do ?

    This method is used to register callbacks. 

    Sometimes, we need to be notified when two or more tag points meet the conditions , so this method is needed.

    The callbacks should be registered by  "RegisterMultiConditionAsyncCallBack“ 。 

    for example :

    ```c#
        public override void RegisterActions()
        {
            RegisterMultiConditionAsyncCallBack(
                OnFinished.BuildAction()
                    .DependenceOn(Tags.SystemRunning, true)
            );
    
        }
    
        public event Action OnFinished;
    ```

 the code shows : 

 when tag point "Tags.SystemRunning” comes to  "True" , then event action "OnFinished" will be called asynchronous.


 4.How to define tags 

 Tags should be defined inside a class which is marked with "[OpcProject]"

 for example :

```c#
[OpcProject(" project description")]
class Tags
{
    public static OpcTagItem<bool> StartSys { get; } = new OpcTagItem<bool>(TagsGenerator.Generate("AnXTQD"), "tag point description");
    
    public static OpcTagItem<int> CurrentPoint { get; } = new OpcTagItem<int>(TagsGenerator.Generate("cDqCyPoint"), "当前采样点");
	
    public static IndexedOpcTagItem<double> SetPointX { get; } = new IndexedOpcTagItem<double>(TagsGenerator.Generate("DcZb"), "某个采样点的 x 坐标", new[] { 18 });
}
```

the code shows:  

  a tags class defined , which is named "Tags" , and is marked with  [OpcProject(" project description")]. 

  a tag is an  “OpcTagItem<T>”  or "IndexedOpcTagItem<T>".

     the difference is "IndexedOpcTagItem" uses numbers as tags count.

     the "IndexedOpcTagItem" argument "indexes" is the tags count.  it should be like bellows :

  1. only one number .

     for example , 18 . it means , you will add 18 tag points , that the index is from 0  to 17 .

  2. two number

     we will find out the min value and the max value , then add tags from min to max

  3.  others

     it means , we will add tags with each number 

     for example : [3,7]

         we will add Tag3 and Tag7.


5. How to write a value 

   IndexedOpcTagItem write value using "public void Write(int index, T value)"

   Switch opc tag write value using  "public void WriteSwitch(bool isReverse = false, int interval = 200, bool isOnlyOn = false)"

   WriteSwitch will write 1 , add then 0. if "isReverse " is set to "true" , it will write 0 first .

   if "isOnlyOn "  is set to true , it will just write 1 or 0 only. 
