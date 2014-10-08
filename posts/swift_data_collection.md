---
title: iOS device motion data collection using Swift
author: Carlos Alonso
date: "2014-10-08"
description: How MyDrive Solutions collects iOS device motion data using Swift and how we managed to avoid running out of memory
tags: ["iOS", "Swift", "memory management", "data collection"]
categories : ["development", "programming"]
---

Here at MyDrive we are completely data-driven and as such we need to collect
as much data as possible. For that purpose we have designed and created our
own iOS data collection app (a.k.a. the iOS awesome app) that silently records
all our movements (lovely, isn't it?).

Now let's get to the technical part. The application has been written in Apple's
brand new programming language: Swift, and we started developing it since it was
in beta version (sorry, I cannot remember exactly which one), but that means
that parts of our codebase had to be rewritten to adopt the incoming changes
from any new released beta.

That has been my first hands-on time with Swift and I have to say that I like it.
Although I used to like it more in one of the betas than in the current first release.
The idea of writing

```swift
if var v = some_method() {

}
```
or
```swift
if some_var? {

}
```

rather than:

```swift
var v = some_method()
if v != nil {

}
```

was, IMO, really a cleaner way of doing nil variable testing, but, for some reason
Apple decided to remove that feature.

Apart from that detail, I think that the language has some 'musts' and cool features
as closures support, tuples and multiple return values, variable types inference and
functional programming patterns and others not so cool like optionals, dodgy
casting errors, different parameters names from within or outside of the function
or the fact that it is still very young and things are likely to change in the short/mid term.

Finished giving my first impressions on Swift, let's now move on to the so called
`ios awesome app` and particularly to the most technically interesting part of
it that is how we managed to be able to record accelerometer data for hours
without running out of memory, because, as you have guessed, our first approach
was to store all the accelerometer observations in an array structure and then,
when finished recording, dump it for gzipping and submitting to the cloud storage.

That 'all in memory/brute force' approach worked not bad for a while, given that
we were collecting data at 1Hz frequency, but when we required to start recording
data at higher frequencies (30 and 60 Hz), problems soon appeared.

After spotting the cause of the issue, we decided to create a custom `NSOperation`
meant to run outside the main `NSOperationQueue` that, from time to time simply
dumps the contents of the array holding the accelerometer data to a file in disk
through a `NSOutputStream` and that worked fine except for the fact that after some
time using the app we realised that the last batch wasn't fully dumped
(failed to wait for the dumping queue to finish before reading for gzipping).

Once solved, the code looks more or less like this:

```swift
func addDataRow(...) {
  data.append(...)

  if data.count >= 300 {
    let toBeDump = data
    data = [Row]()
    dumpArray(toBeDump)
  }
}

func dumpArray(data: [Row]!) {
  dumpQueue.addOperation(CSVDumpOperation(file: filePath, data: data))
}
```

This piece of code simply appends data to an array and when it's reaches a
size, copies it to another variable and clears it to avoid it becoming too large.
The copy is then given to a custom `NSOperation` to be dump to disk outside the
main operation queue.

The dump operation looks like this:

```swift
class CSVDumpOperation: NSOperation {

  let data = [Row]()
  let os : NSOutputStream

  init(file: String, data: [Row]) {
    os = NSOutputStream(toFileAtPath: file, append: true)
    os.open()

    super.init()

    self.data = data
  }

  override func main() {
    for row in data {
      let rowStr = "\(row.x),\(row.y),\(row.z)...\n"
      if let rowData = rowStr.dataUsingEncoding(NSUTF8StringEncoding, allowLossyConversion: false) {
        let bytes = UnsafePointer<UInt8>(rowData.bytes)
        os.write(bytes, maxLength: rowData.length)
      }
    }

    os.close()
  }
}
```

This `CSVDumpOperation` simply opens a `NSOutputStream` to the file and writes there
the csv formatted contents of the given array.

And that's it!, with this simple approach for this simple application we intend
to collect hundreds of hours of different activities for further analysis.
