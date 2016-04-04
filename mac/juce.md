//==============================================================================================

## JUCE

#### XCode debug
    {(const char*) $VAR.text.data}:s
    {(const char*) $VAR.fullPath.text.data}:s

    https://github.com/open-ephys/GUI.git

    JUCE_DEBUG
    JUCE_ASIO_DEBUGGING
    JUCE_ENABLE_REPAINT_DEBUGGING
    ignoreUnused
    JUCEApplication::getCommandLineParameters()
    forEachXmlChildElement // define
    juce::DeletedAtShutdown
    juce_DeclareSingleton (Class, true)
    juce::SystemTrayIconComponent
    numElementsInArray(arr) // const int arr[] = { 1,2,3,4,7 };

if component methods are being called from threads other than the message  
thread, you'll need to use a MessageManagerLock object to make sure it's thread-safe.

    ASSERT_MESSAGE_MANAGER_IS_LOCKED_OR_OFFSCREEN
    callFunctionOnMessageThread
    (new CallbackMessage())->post();

#### ConsoleLogger
    class ConsoleLogger : public Logger {
        void logMessage (const String& message) override {
            std::cout << message << std::endl;
        }
    };
    ConsoleLogger logger;
    Logger::setCurrentLogger (&logger);
    Logger::writeToLog("hello");
    Logger::setCurrentLogger (nullptr);

#### Arrays
**Array** used to hold simple, non-polymorphic objects as well as primitive types  
**OwnedArray** holds a list of pointers to objects, and will automatically delete the objects when  
they are removed from the array, or when the array is itself deleted  
**ReferenceCountedArray** holds objects derived from *ReferenceCountedObject*, and takes care of  
incrementing and decrementing their ref counts when they are added and removed from the array  
**StringArray** special array for holding a list of strings  
**StringPairArray** container for holding a set of strings which are keyed by another string  

#### AudioFormatReader
Reads samples from an audio file stream.  
A subclass that reads a specific type of audio format will be created by an *AudioFormat* object  
**AudioCDReader** used to read from an audio CD as if it's one big audio stream  
**AudioSubsectionReader** if you have a reader which can read a 1000 sample file, you could wrap it  
in one of these to only access, e.g. samples 100 to 200, and any samples outside that will come  
back as 0. Accessing sample 0 from this reader will actually read the first sample from the  
other's subsection, which might be at a non-zero position  
**BufferingAudioReader** uses a background thread to pre-read data from another reader  
**MemoryMappedAudioFormatReader** specialised type to read directly from an audio file, allows for  
incredibly fast random-access to sample data in the mapped region of the file

#### AudioSource
Base class for objects that can produce a continuous stream of audio  
**AudioAppComponent** A base class for writing audio apps that stream from the audio i/o devices.  
provides a basic *AudioDeviceManager* object and runs audio through the default output device
**ChannelRemappingAudioSource** takes the audio from another source, and re-maps its input and  
output channels to a different arrangement. You can use this to increase or decrease the number  
of channels that an audio source uses, or to re-order those channels  
**IIRFilterAudioSource** performs an IIR filter on another source  
**MixerAudioSource** mixes together the output of a set of other *AudioSources*  
**PositionableAudioSource** can be repositioned, has a current read position  
**ResamplingAudioSource** takes an input source and changes its sample rate  
**ReverbAudioSource** uses the *Reverb* class to apply a reverb to another *AudioSource*  
**ToneGeneratorAudioSource** generates a sine wave  
**AudioTransportSource** takes *PositionableAudioSource* and allows it to be played, stopped, started
This can also be told use a buffer and background thread to read ahead, and if can correct for  
different sample-rates. You may want to use one of these along with an *AudioSourcePlayer* and  
*AudioIODevice* to control playback of an audio file  
    start() stop()
    getLengthInSeconds()
    setPosition(double newPosition) getCurrentPosition()
**AudioFormatReaderSource** read from an *AudioFormatReader*  
**BufferingAudioSource** takes another source as input, and buffers it using a thread.  
Create this as a wrapper around another thread, and it will read-ahead with a background thread  
to smooth out playback.    
You can create one of these directly, or use it indirectly using an *AudioTransportSource*  
AudioSourceChannelInfo -- AudioSampleBuffer, i startSample, i numSamples

#### AudioSampleBuffer
    getNum Samples/Channels
    get Read/Write Pointer
    add/get/set/ Sample; clear
    copyFrom/addFrom/applyGain	+ WithRamp
    getRMSLevel; getMagnitude; findMinMax; reverse

#### Formats
**AudioFormatManager**  
**AudioFormatWriter**  

*ThreadWithProgressWindow*  

#### Visualize
*AudioVisualiserComponent*  
or  
*AudioThumbnail* + *AudioIODeviceCallback*  

#### tracks
    sndfile-info.app
    AudioFormatReader::usesFloatingPointData
    AudioFormatReader (false in ctor)
    BufferingAudioReader (true in ctor)
    WavAudioFormatReader (true in ctor in case:
        format == 3
        format == 0xfffe /*WAVE_FORMAT_EXTENSIBLE*/
        subFormat == IEEEFloatFormat
    )
    
    AsyncUpdater - handleAsyncUpdate()=0;
    ChangeBroadcaster
    ChangeBroadcasterCallback: public AsyncUpdater
    ChangeListener

#### Usage examples
    DBG("hello kittie!");
    Logger* log = Logger::getCurrentLogger();
    String msg("log message");
    msg = msg + String(": details ");
    msg << 42 << " at ";
    
    Time now = Time::getCurrentTime();
    msg << now.toString(true, true, true, true);

    //File file("./test_file.txt"); // same as
    File file = File::getSpecialLocation(File::currentExecutableFile).getParentDirectory().getChildFile("./test_file.txt");
    
    file.replaceWithText(msg);
    
    String text = file.loadFileAsString();
    log->writeToLog(text);

    FileInputStream fis(file);
    if (fis.openedOk()) {
        log->writeToLog("as stream: " + fis.readEntireStreamAsString());
    }
    
    log->writeToLog("cre: " + file.getCreationTime().toString(true, true, true, true));
    log->writeToLog("mod: " + file.getLastModificationTime().toString(true, true, true, true));
    log->writeToLog("acc: " + file.getLastAccessTime().toString(true, true, true, true));
    
    File root = File::getSpecialLocation(File::userDocumentsDirectory);
    File dir1 = root.getChildFile("t1");
    File dir2 = dir1.getChildFile("t2");
    Result res = dir2.createDirectory();
    if (res.wasOk()) {
        log->writeToLog(dir2.getFullPathName() + String(" created"));
        log->writeToLog(dir2.getRelativePathFrom(file) + String(" to test_file.txt"));
    } else {
        log->writeToLog("fail");
    }
    
    Array<File> resFiles;
    dir1.findChildFiles(resFiles, File::findFilesAndDirectories, false);
    for (int i=0; i<resFiles.size(); ++i) {
        log->writeToLog(resFiles[i].getFullPathName());
    }
    
    StringArray strs;
    strs.addTokens("one two three", true);
    for (int i=0; i<strs.size(); ++i) {
        log->writeToLog(strs[i]);
    }

    void print_progress(float x) {
    	int percent = (int)(100*x);
    	std::cout << "\r" << std::setw(3) << percent << "% completed ";
    		<< "[" << std::string(percent/2, 'I')
    		<< std::string((50 - percent/2), '.') << "]";
    	std::cout.flush();
    	if (percent == 100) std::cout << std::endl << "Operation completed successfully." << std::endl;
    }

    private:
        AudioDeviceManager deviceManager;
        AudioDeviceSelectorComponent deviceComponent;
        AudioProcessorGraph graph;
        AudioProcessorPlayer graphPlayer;
    
    MainContentComponent::MainContentComponent()
        : deviceComponent(deviceManager, 0, 2, 0, 2, false, false, true, false)
    {
        addAndMakeVisible(deviceComponent);
        
        deviceManager.initialise(2, 2, nullptr, true);
        graphPlayer.setProcessor(&graph);
        deviceManager.addAudioCallback(&graphPlayer);
        
        AudioProcessorGraph::Node* inNode = graph.addNode(
          new AudioProcessorGraph::AudioGraphIOProcessor(
            AudioProcessorGraph::AudioGraphIOProcessor::audioInputNode));
        AudioProcessorGraph::Node* outNode = graph.addNode(
          new AudioProcessorGraph::AudioGraphIOProcessor(
            AudioProcessorGraph::AudioGraphIOProcessor::audioOutputNode));
        
        graph.addConnection(inNode->nodeId, 0, outNode->nodeId, 0);
        graph.addConnection(inNode->nodeId, 1, outNode->nodeId, 1);
        
        setSize (600, 400);
    }
    
    MainContentComponent::~MainContentComponent()
    {
        deviceManager.removeAudioCallback(&graphPlayer);
        graphPlayer.setProcessor(nullptr);
        deviceManager.closeAudioDevice();
    }

#### Resizing
    void MainContentComponent::resized() {
        int heigth = getHeight();
        int width = getWidth();
    
        //mDeviceComponent.setBounds(0,0, 330,headerHeigth);
        int y = 0;
        for (int i = mDeviceComponent.getNumChildComponents(); --i >= 0;)
            y = jmax (y, mDeviceComponent.getChildComponent (i)->getBottom());
        Rectangle<int> area (getLocalBounds());
        y += 50;
        area.setBottom (y);
        area.setWidth(jmin(area.getWidth(), 330));
        mDeviceComponent.setBounds (area);
    
        area = getLocalBounds();
        area.setTop (y);
        recorderControl.setBounds (area.reduced (8));
    }

#### ComponentBuilder

    class View
            : public Component
            , public Button::Listener
            , public TextEditor::Listener
            , public ActionBroadcaster
    {
    public:
        View()
        {

        }

        View(ActionListener * listener)
        {
            addActionListener(listener);
        }

        virtual ~View()
        {
            deleteAllChildren();
        }

        void paint (Graphics& g)
        {
            g.fillAll (Colour (0xff6d8579));
        }

        void resized()
        {

        }

        virtual void buttonClicked (Button* buttonThatWasClicked) {
            sendActionMessage (buttonThatWasClicked->getComponentID());
        }
    };

    template <class ComponentClass>
    class ComponentTypeHandler
            : public ComponentBuilder::TypeHandler
    {
    public:
        ComponentTypeHandler(const Identifier& valueTreeType_)
            : ComponentBuilder::TypeHandler (valueTreeType_)
        {
        }

        Component* addNewComponentFromState (const ValueTree& state, Component* parent)
        {
            Component* const c = new ComponentClass();

            if (parent != nullptr)
                parent->addAndMakeVisible (c);

            updateComponentFromState (c, state);
            return c;
        }

        void updateComponentFromState (Component* component, const ValueTree& state)
        {
            ComponentClass* const c = dynamic_cast <ComponentClass*> (component);
            jassert (c != nullptr);

            ValueTree babbo = state.getParent();
            int myIndex = babbo.indexOf(state);

            c->setBounds (72, myIndex*24+5, 150, 24);
            c->setText (state.getProperty("value"));
        }
    };

    template <>
    class ComponentTypeHandler<TextButton>
            : public ComponentBuilder::TypeHandler
    {
    public:
        ComponentTypeHandler()
            : ComponentBuilder::TypeHandler ("TextButton")
        {
        }

        Component* addNewComponentFromState (const ValueTree& state, Component* parent)
        {
            TextButton* const b = new TextButton();

            if (parent != nullptr)
            {
                parent->addAndMakeVisible (b);
                b->addListener(dynamic_cast<Button::Listener*>(parent));
            }

            updateComponentFromState (b, state);
            return b;
        }

        void updateComponentFromState (Component* component, const ValueTree& state)
        {
            TextButton* const b = dynamic_cast <TextButton*> (component);
            jassert (b != nullptr);

            ValueTree babbo = state.getParent();
            int myIndex = babbo.indexOf(state);

            b->setBounds (72, myIndex*24+5, 150, 24);

            b->setButtonText (state.getProperty("value"));
        }
    };

    template <>
    class ComponentTypeHandler<View>  : public ComponentBuilder::TypeHandler
    {
    public:
        ComponentTypeHandler()
            : ComponentBuilder::TypeHandler ("SimpleForm")
        {
        }

        Component* addNewComponentFromState (const ValueTree& state, Component* parent)
        {
            Component * c = new View();

            if (parent != nullptr)
                parent->addAndMakeVisible (c);

            updateComponentFromState (c, state);

            getBuilder()->updateChildComponents(*c, state);

            return c;
        }

        void updateComponentFromState (Component* component, const ValueTree& state)
        {
            View* const c = dynamic_cast <View*> (component);
            jassert (c != nullptr);

            c->setSize (800, 600);
        }
    };

    void registerComponentTypeHandlers (ComponentBuilder& builder)
    {
        builder.registerTypeHandler (new ComponentTypeHandler <TextEditor>("TextEditor"));
        builder.registerTypeHandler (new ComponentTypeHandler <TextButton>());
        builder.registerTypeHandler (new ComponentTypeHandler <View>());
    }


    class Controller : public ActionListener
    {
    public:

        Controller()
            : cb (values)
        {
            initValues();

            //cb.state = values;

            registerComponentTypeHandlers(cb);

            ActionBroadcaster * view = dynamic_cast<ActionBroadcaster*> (cb.getManagedComponent());

            view-> addActionListener(this);
        }

        virtual ~Controller()
        {

        }

        virtual void actionListenerCallback (const String& message)
        {
            AlertWindow::showMessageBox (AlertWindow::InfoIcon,
                                         message + " clicked",
                                         values.toXmlString()  );

            Logger::writeToLog(values.toXmlString() );

            values.getChildWithProperty("id", "Pippo").setProperty("value", 42, nullptr);
        }

        Component * getComponent ()
        {
            return cb.getManagedComponent();
        }

    private:
        ValueTree values;
        ComponentBuilder cb;

        void initValues()
        {
            String xmlString (
                        "<SimpleForm id=\"MyForm0001\">"
                        "<TextEditor id=\"Pippo\" value=\"123\"/>"
                        "<TextEditor id=\"Pluto\" value=\"AAA\"/>"
                        "<TextButton id=\"Carlo\" value=\"OK\"/>"
                        "</SimpleForm>");

            ScopedPointer<XmlElement> element = XmlDocument::parse(xmlString);

            values = ValueTree::fromXml(*element);

        }
    };



