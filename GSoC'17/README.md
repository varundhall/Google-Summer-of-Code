# Replace EditEngine binary clipboard with ODF filter

## Organization : LibreOffice

## Abstract
[EditEngine](https://docs.libreoffice.org/editeng.html) uses a legacy [StarOffice](https://en.wikipedia.org/wiki/StarOffice) format with SfxItemPool binary serialization for copy/paste which is a maintainability problem and the only user of SfxItemPool serialization outside of the table auto-formats. It should be replaced by using the [ODF](http://opendocumentformat.org/) filter ([xmloff](https://cgit.freedesktop.org/libreoffice/core/tree/xmloff)) for copy/paste instead. The legacy format is used if we edit inside some text in [Draw](https://www.libreoffice.org/discover/draw/)/[Impress](https://www.libreoffice.org/discover/impress/)/[Calc](https://www.libreoffice.org/discover/calc/) and copy a selection to the clipboard.


## Project Goals
Replacing the legacy format for copy/paste operations in LibreOffice, by using ODF filter instead of SfxItemPool binary serialization. Adding of new unit tests to prevent regressions.

## Background : LibreOffice and EditEngine
[LibreOffice](https://www.libreoffice.org/) is a freely available, fully-featured office productivity suite. Its native file format is OpenDocument, an open standard format that is being adopted by governments worldwide as a required file format for publishing and accepting documents. LibreOffice can also open and save documents in many other formats, including those used by several versions of Microsoft Office.

[EditEngine](https://docs.libreoffice.org/editeng.html) is a primary module within LibreOffice source-code which is capable of performing various text related operations, it offer certain features for text related operations. It is often used inside many applications like Draw/Impress/Calc/Writer for text related operations. It uses some legacy StarOffice techniques and old formats to achieve the tasks and to provide diverse functionality.


## GSoC Coding phase overview
I started the project by writing few unit tests for EditEngine that uses the legacy format for copy/paste operations, so that they things can be tested after format change.

Then I added a new identifier `SotClipboardFormatId::EDITENGINE_ODF_TEXT_FLAT` for the new clipboard format in generic (platform independent) clipboard code. Earlier  `SotClipboardFormatId::EDITENGINE` was used for Legacy format.

After adding the identifier, I modified various EditEngine's clipboard methods to change the format that is currently used as they initialize the import/export filter code. During this process I also made few changes to `xmloff` to adjust the import/export filter for replacement.

Then I made changes in all the application's code (Draw/Impress/Calc) to use the new copy/paste format. I also made few changes in `dtrans` to use the new format in Windows platform. 

After that I inspected the new clipboard format, made some tweaks for selection and hyperlinks copy/paste. Wrote couple of new unit tests to examine everything is working correctly or not.

Once all the changes were in master, I dropped the notification to LibreOffice Q/A team for further examination, luckily no new bugs were found and I proceeded with removal of old format.

I again modified couple of EditEngine clipboard methods and after that removed all the unused SfxItemPool serialisation code from all the applications and other modules.


## Major achievements of the project
 - StarOffice format is not used anymore for copy/paste operations
 - SfxItemPool serialisation is not used anywhere outside Table Auto-Formats
 - Using ODF filter text is copied with proper selection
 - Hyperlink fields are copied properly without any loss of information
 - All the unused serialisation code is removed from all the modules and applications, which solves the maintainability problem


## List of all my changes
Following is the list of all patches written by me during entire GSoC timeline. They can be viewed under `Patches` sub-directory or on [cgit](https://cgit.freedesktop.org/libreoffice/core/log/?qt=author&q=varun) or on [GitHub](https://github.com/LibreOffice/core/commits?author=varundhall)

- 5ae5fb482f37176f1746cca4ade5c87b34b135a7 EditEngine Refactoring hand-coded XInterface implementations of EditDataObject
- 33e53bf634b629357dd643a900d3969ad2622510 EditEngine: Added ODF_TEXT_FLAT copy for sc
- 223a04e77fe9702eb167e69165ec57c46512a845 EditEngine Exporting flat XML from actual clipboard
- 0703af9e782f1e03c243ee886d1301e80817aad8 Exporting EditEngine document into a buffer with ODF Filter
- b863b1cb9c72d04933e6f9d3c361b8ebf9399cfd EditEngine Implementing Paste for XML from clipboard
- fec801d6d980f9bd277051ec07b55b8f2845ccc0 EditEngine Renamed InsertText to PasteText
- 65f98346c2edc983421643a7f5c32313b45934e0 EditEngine Creating SvXMLImportContext for ODF
- b0f0096fbbccc212931501c5c879b65f9b0d7477 EditEngine: Added ODF_TEXT_FLAT paste for sc
- 112a82566453fcf85ce3619965ca6e11fa9d29c5 EditEngine: Added ODF_TEXT_FLAT copy/paste for sd
- 3278cbf7dcf0c72c125fb4ba14e2b24faf267d9c EditEngine: Added EditEngine ODF format for Windows
- 84284429de635226342d745680fa5ddc324b4b3b EditEngine: Added test to check Multi Para Copy/Paste
- 41f1474e63a40e682a424211e788cc4b801e6659 EditEngine: Added test to check MultiPara SelectiveSelection Copy/Paste
- 1ace4ed5c7152e7ac81f07e86d794e83b1cebc2c EditEngine: Added test to check MultiPara Copy/Paste with Bold/Italic text
- 3c1fc723ff622d8a541fa26a3397ca4258332e4a EditEngine: Changing export selection for copy/paste
- 6e7300d1046a195068fa97a0d91a95f19cc5c056 EditEngine: Added test to check multi para start copy/paste
- 2de270e4fd0ab96691b251b38caf71b3fc87574d EditEngine: Added test for large para copy/paste
- a62c9ecfc27a375cddb2beefba4b5af9bd4f7727 EditEngine: Added warning for xml export exception
- 9479171a09ba4c73afa8b40a5c2590df3b6d5415 EditEngine: Added para break/connect during ODF paste
- 7878bcf5547ae4d111aa8e835529e501c4e23838 xmloff: Updated XMLTextListBlockContext to insert elements before NumRules
- 45fdb40085f5090ccd8df23f461ef9a8269d11b2 EditEngine: Added SvxSimpleUnoModel in import filter
- 35ba68fd471d7d2f50183a1ffb7348b1b069616d EditEngine: Added expwarp for unit tests
- e333183d4390da0b17a55f214e2b953dbb0a8883 EditEngine: Making ODF Format Default for Copy/Paste
- 315b6afc90298ac23de7d24449feae6beaade17a Partially revert c1723a3b6
- e0bafa78e3ad0df397d78cd65ad19bd5b07dc5f2 Added Test to check TableAutoFormat Style after Save/Load
- 9fa77d0cf0dbc0bcb627848263c417ba49e988d3 EditEngine: Removing BIN format from sc
- 8a8bcd65e8a0e177904f06fcd8a775456ff02eab EditEngine: Removing BIN format from sd
- c0998d9481cd57560a4d549d760f0be9da510306 EditEngine: Removing BIN format for Windows
- 4aef9f5e330a4228b2355fa97900dcede3e375c4 EditEngine: Removing BIN format
- 544a978132dd8f9d44756f62f160a0883193d54a EditEngine: Removing BIN format from sot
- d10a61fe36760336842cbfa7f8f77e1fb9a475a5 Removing unused SfxItemPool serialisation from sfx2
- 1e4b1e4a1aca3b333820b0a865997e6f62e80064 Removing unused SfxItemPool serialisation from editeng
- 9cc75c52369d6656a5812e950641d4b9b6d0cc35 Removing unused SfxItemPool serialisation from sc
- 97b889b8b2b2554ce33fd6b3f0359fc18f39832d WIP: Removing unused SfxItemPool serialisation from svx
- 8b3fcb6a4f80e803586120535768745f6637b34d Removing unused SfxItemPool serialisation from svx
- 38984e9351a0230b0a56fb7277f7b19722350d8d Removing unused SfxItemPool serialisation from svx
- 78b3e279ca3a3da1e7a49756bc1edee82fc943a0 Removing unused SetVersionMap from sc
- cf9a90c60f4bedd94d72465b13449f2ccf089d28 Removing unused SetVersionMap from sw
- ec2abbca3d9befc4192452555a9433d267a47d58 Removing unused SetVersionMap from editeng
- 57db6e24b5ad43d447c30e44a112c74c7e75b46b Removing old SfxItemSet::getHash usage
- 5be19d80f47cca8260cb46ac6e54ac19e5ea2705 Replace stringify by Equals in SdrObject and SdPage
- b021353dd62c3d8c9ee0281753b88f6304a2514d Removing unused serialisation code
- bc11bee358a3bcdf49ef60ee0449bcd053c00d74 Removing unused serialisation code from svl
- 75933b220d48bceff25b07cfc4b55c70a2e24917 Rename SfxItemPool GetSurrogate to CheckItemInPool
- 006a7b50546c57e260245d4630de565705f2fc38 Removing unused serialisation code