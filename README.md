<div align="center">

## Guitar Note Recognizer


</div>

### Description

A study about the structure of audio format and digital musical notes is also required in order to accomplish the project. The aim of this project is to produce a program that will analyze a music track and output the guitar notes that were played on the track. This Microsoft Windows platform software application will be able to listen for acoustic guitar sounds from tracks where only one guitar is being played. The data will be outputted in tab notation. By creating this software, the public especially guitar players can get the guitar notes directly from the audio file (WAV format), probably from a Compact Disc without wasting money to buy music book from the shop or search the scores from the internet.

Please visit website::

http://softworld.filetap.com
 
### More Info
 


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Mecoli](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/mecoli.md)
**Level**          |Advanced
**User Rating**    |5.0 (10 globes from 2 users)
**Compatibility**  |C\+\+ \(general\), Microsoft Visual C\+\+, Borland C\+\+, UNIX C\+\+
**Category**       |[Complete Applications](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/complete-applications__3-7.md)
**World**          |[C / C\+\+](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/c-c.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/mecoli-guitar-note-recognizer__3-6737/archive/master.zip)





### Source Code

```
//Below is my one of the guitar note generator
//program, to get the source code pls go my
//website
//http://softworld.filetap.com
//
#include "stdafx.h"
#include "aunew.h"
#include "mainfrm.h"
#include "NoteGenerator.h"
#include "winex.h"
#ifdef _DEBUG
#undef THIS_FILE
static char THIS_FILE[]=__FILE__;
#define new DEBUG_NEW
#endif
//////////////////////////////////////////////////////////////////////
// Construction/Destruction
//////////////////////////////////////////////////////////////////////
NoteGenerator::NoteGenerator(CNoteFre *spec, CAuNewView *tmp, int samplesPerBuf,int samplesPerSec,
							 int fftPoints)
							 :
	_specView(spec),
	_wavView(tmp),
	_samplesPerBuf (samplesPerBuf),
  _samplesPerSecond (samplesPerSec),
  _fftPoints (fftPoints),
	_bitsPerSample(16),
  _pFftTransformer (fftPoints, samplesPerSec),
  _pListener (samplesPerBuf, samplesPerSec),
	_isActive(0),
	_listenMode(0)
{
}
NoteGenerator::~NoteGenerator()
{
}
//Starts a thread and waits for command to resume
void NoteGenerator::Run()
{
	//Infinite loop waiting
  for (;;)
  {
    _event.Wait ();
    if (_isDying)
      return;
		//Lock makes sure control remains with current thread
    Lock lock (_mutex);
		_isActive = 1;
    if (_pListener->IsBufferDone ())
			//Main control funciton
			LokWaveInData ();
  }
}
//Re-initializes the Listener
BOOL NoteGenerator::ReInit(int samplesPerBuf,
    int samplesPerSec,
    int fftPoints,
    int bitsPerSample)
{
  Lock lock (_mutex);
  if (_pListener->BitsPerSample() == bitsPerSample &&
    _pListener->SamplesPerSecond() == samplesPerSec &&
    _pFftTransformer->Points() == fftPoints &&
    _pListener->SampleCount() == samplesPerBuf)
  {
    return TRUE;
  }
  _samplesPerBuf = samplesPerBuf;
  _samplesPerSecond = samplesPerSec;
  _fftPoints = fftPoints;
  _bitsPerSample = bitsPerSample;
  BOOL isStarted = _pListener->IsStarted();
  if (isStarted)
    _pListener->Stop();
  BOOL isError = FALSE;
  try
  {
    _pFftTransformer.ReInit ( _fftPoints, _samplesPerSecond);
    _pListener.ReInit (_bitsPerSample, _samplesPerBuf, _samplesPerSecond);
  }
  catch ( WinException e )
  {
    char buf[50];
    wsprintf ( buf, "%s, Error %d", e.GetMessage(), e.GetError() );
    MessageBox (0, buf, "Exception", MB_ICONEXCLAMATION | MB_OK);
    isError = TRUE;
  }
  catch (...)
  {
    MessageBox (0, "Unknown", "Exception", MB_ICONEXCLAMATION | MB_OK);
    isError = TRUE;
  }
  if (isStarted)
  {
    isError = !_pListener->Start(_event);
  }
  return !isError;
}
void NoteGenerator::LokWaveInData()
{
	//An iterator class used to access the Listener data
	SampleIter iter (_pListener.GetAccess());
  // Quickly release the buffer
  if (!_pListener->BufferDone ())
    return;
  //Copy the data into the FFT buffer and perform transform on it
	_pFftTransformer->CopyIn (iter);
  _pFftTransformer->Transform();
	//Update the spectrograph
	char ret[3];
	_specView->Update (_pFftTransformer.GetAccess(), ret);
}
BOOL NoteGenerator::Start ()
{
	Lock lock (_mutex);
  _isActive = 1;
	//Start the Listener recording from the soundcard
	return _pListener->Start (_event);
}
void NoteGenerator::Stop ()
{
	Lock lock (_mutex);
  _isActive = 0;
	//Stop it recording
	_pListener->Stop ();
}
//That is my one of the guitar note generator
//program, to get the complete source code, or for more information pls go my website
//http://softworld.filetap.com
```

