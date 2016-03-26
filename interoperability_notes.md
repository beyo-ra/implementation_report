# PreForma Supplier Conformance XML

This documentent is originally drafted by Dave Rice but intended to serve for collaboration purposes amongst suppliers to negotiate, define, and work towards shared standards for expressing the results of implementation checks where feasible.

## General Structural Comparison

At a high level XML output from the 3 suppliers all followed this general structure.

```xml
<reports_level_name>
    <report_about_one_file>
        <technical_metadata_about_a_file/>
        <validation_results_about_a_file/>
    </report_about_one_file>
</reports_level_name>
```

In Verapdf's case this is expressed like:
```xml
<cliReprot>
    <itemDetails/>
    <validationResult>
        <assertions>
            <assertion/>
        </assertions>
    </validationResult>
</cliReport>
```

For DPF Manager:
```xml
<globalreport>
    <individualreports>
        <report>
            <!-- technical details here -->
            <implementation_checker>
                <!-- validationresults per implementation type -->
                <results>
                    <!-- validationresults of all implementation type together -->
                </results>
            </implementation_checker>
        </report>
    </individualreports>
    <stats/>
</globalreport>
```

From MediaConch:
```xml
<MediaConch><!-- in the case of MediaConch the technical metadata goes inanother XML expression that may neighbor <MediaConch> -->
    <media>
        <implementationChecks>
            <check/>
        </implementationChecks>
    </media>
</MediaConch>
```

Notes:
    I'm unclear how a Verapdf report can report on more than one file in a single document.

## Proposal for Normalization

### results of implementation checker validation

To prioritize conformance these expressions of conformance I recommend focusing on the section that expression the results of an implementation test, in other words:
- verapdf:validationResult
- dpfmanager:implementation_checker
- mediaconch:implementationChecks

### file identity

Beyond this there are some other aspects to standardize. It should be clear how to identify the file in each report in a consistent way. Presently we have:
- verapdf:cliReport/itemDetails/name
- dpfmanager:globalreport/individualreports/report/file_info/name
- mediaconch:MediaConch/media/@ref

### overall result

I also propose that the general pass/fail/not-applicable be easy to parse in a consistant way from our reports. Presently that is:
- verapdf:cliReport/validationResult/@isCompliant
- globalreport/individualreports/report/implementation_checker/results/result/level ??
- derived from mediaconch:MediaConch/media/implementationChecks/check/test/@outcome

## implementation_checker

I propose normalization to Preforma's vocabulary for
- verapdf:validationResult
- dpfmanager:implementation_checker
- mediaconch:implementationChecks

By merging MediaConch and VeraPDF we could have this structure:
```xml
<implementation_checker ref="" format="" flavour="" version="" totalAssertions="" isCompliant="">
    <assertions tobedefined="in another section of this doc"/>
    <implementation_checker/><!-- possible recursion -->
</implementation_checker>
```

To VeraPdf this adds attributes for `@format` and `@version` and a `name` and `description` sub-element. I propose name to hold a human readable name of the implementation checker tool, so that one could have:
```xml
<report>
    <implementation_checker ref="dpfmanager.org" format="TIFF">
    </implementation_checker>
    <implementation_checker ref="tiffcheck.org" format="TIFF">
    </implementation_checker>
</report>
```
or
```xml
<implementation_checker ref="mediaconch.org/ebml" format="EBML">
    <implementation_checker ref="mediaconch.org/mkv" format="Matroska">
    </implementation_checker>
</implementation_checker>
```

Notes:
I'm not sure if `flavour` is the best format-generic way to refer to a profile or sub-category or type of a particular file format. Suggestions?
Is isCompliant a boolean?

## assertions
I have a preference for verapdf's language here, so am switching MediaConch's `check` to `assertion`.

I'm not certain how `@ordinal` is able to cross reference to anything but left it in.

There's a difference between how mediaconch and verapdf/dpfmanager handle assertions. In MediaConch we have a `check` which is a generic rule (for example: that a named element has a valid parent element) and then that `check` uses a subelement `test` to note the outcome of that check on every element. Thus a check of a rule on a file can result in many, many tests. In verapdf the concepts of `test` and `check` are merged. I prefer maintaining the distinct but please advice.

Here's examples of the current assertions:

dpfmanager
```xml
<result>
  <level>critical</level>
  <msg>Missing required tag for TiffEP ImageDescription</msg>
</result>
```
verapdf
```xml
<assertion ordinal="384884" status="FAILED">
    <ruleId specification="ISO_19005_1" clause="6.2.4" testNumber="3"/>
    <message>If an Image dictionary contains the Interpolate key, its value shall be false</message>
    <location>
        <level>CosDocument</level>
        <context>root/document[0]/pages[12]/contentStream[0]/operators[125]/xObject[0]</context>
    </location>
</assertion>
```
mediaconch
```xml
<check icid="MKV-SEEK-RESOLVE" version="1">
    <context field="Offset of First Segment Element" value="59"/>
    <test outcome="pass">
        <value offset="68" name="SeekID">0x1549A966</value>
        <value offset="75" name="SeekPosition">223</value>
    </test>
    <test outcome="pass">
        <value offset="82" name="SeekID">0x1654AE6B</value>
        <value offset="89" name="SeekPosition">302</value>
    </test>
    <test outcome="pass">
        <value offset="97" name="SeekID">0x1254C367</value>
        <value offset="104" name="SeekPosition">436</value>
    </test>
    <test outcome="pass">
        <value offset="112" name="SeekID">0x1C53BB6B</value>
        <value offset="119" name="SeekPosition">108807</value>
    </test>
    <test outcome="fail" reason="The Seek Element at 482161284 octets references an Element with 0x1F43B675 as an ID and a Seek Position of 4645130 but it is not there.">
        <value offset="482161287" name="SeekID">0x1F43B675</value>
        <value offset="482161294" name="SeekPosition">4645130</value>
    </test>
</check>
```

I merged these to create a structure like this:
```xml
<assertion status="FAILED" icid="MKV-SEEK-RESOLVE" version="1">
    <ruleId specification="ISO_19005_1" clause="6.2.4" testNumber="3"/>
    <message>About the assertion</message>
    <context name="Offset of First Segment Element">59</context>
    <test ordinal="384884" outcome="FAILED">
        <reason>The Seek Element at 482161284 octets references an Element with 0x1F43B675 as an ID and a Seek Position of 4645130 but it is not there.</reason>
        <value offset="482161287" name="SeekID">0x1F43B675</value>
        <value offset="482161294" name="SeekPosition">4645130</value>
    </test>
</assertion>
```

The @icid I propose we use as an identifier of the assertion as the implementation checker has interpreted it. I consider it short for `implementation checker identifier`. Happy to change this to simply `id` if preferred.

I like verapdf's location contextualization though in many cases more than one location is assessed to conduct a test.