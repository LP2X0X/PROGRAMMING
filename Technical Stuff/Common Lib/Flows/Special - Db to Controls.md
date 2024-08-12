---

excalidraw-plugin: parsed
tags: [excalidraw]

---
==⚠  Switch to EXCALIDRAW VIEW in the MORE OPTIONS menu of this document. ⚠==


# Text Elements
Main View ^ABuNEKf8

DbConnect View ^esLuTPxE

connectButton ^MKd157BM

ConnectCommand ^Hx8C0gUP

Bind to ^gl4bzUh4

Execute ^Mdsr5x7C

Async ^3TuugcSI

TryConnectDB ^d1iK0Clq

[The Showindicator function helps 
show loading screen
while waiting to connect to DB] ^wPFDQQFl

[ConnectionTest is a function implemented in 
ResultDbReader class] ^IYLGzjfM

DbConnect View ^fQNABkZF

ConnectionTest ^l4gBP0UU

[Try connect to Db with this information] ^MLHxtL5P

CONNECT TO DB ^w76fXodz

POPULATE PCBITEMS DATA GRID ^A2nviXog

Main View ^NGy853j9

DbConnect ^5qSo3opz

pcbRefreshButton ^7qhMRLVE

PcbGridRefreshCommand ^pSN7eGEw

Bind to ^HxCW8nX9

Execute ^isflkmm6

Async ^HMZbfz1W

Showindicator ^IWwIbnzG

RefreshQuery ^Kho0kBB1

Await ^x2XXcTf2

[Clear old data] ^pW4f9idO

Await ^pNQctWC8

ExecuteSelectPCBQuery ^0sU1cMtP

Await ^UCCDntDB

CreateInspectionBoardAsync ^eOKJlUtg

[This is a function implemented in 
ResultDbReader class] ^xEjhknWl

return List<Board> ^3gVEp3RH

            Pass to PCBItems
 (which is source for pcbDataGrid) ^q35YRT2q

ExecuteQueryAsync ^bXsqhlWl

Await ^mPaoEPt5

Dapper ^CjHKtRSL

Using ^pdSB4L3E

return IEnumarable<T> ^pBarH0G1

convert to List ^shkzeax4

return List<T> ^QOQHlxfF

Convert to List<Board> here! ^BUhYa1FO

Invoke ^a0vWRJrr

Execute ^i4awtjXx

Async ^Uw4BUYMU

Showindicator ^HXECpZR2

Await ^pyGEPgMe

Handle ^AI4JHQSE

PCBSelectionChangedCommand ^QxpIHkpk

Pass SelectedItem (board) as para ^ELsUPh1H

SelectedPCBChanged ^ay2axinY

If the board object is null  ^1nERSNAi

If the board object is not null  ^ZadP42a1

If the PCBItems source already got populated
and the selected PCB is already selected once ^R5zTwzuK

If the PCBItems source is empty
or the selected PCB is a newly
selected one ^94PFc95c

GetBoardInfoAsync ^aUAOwCMw

Await ^RC8VcsTK

CreateBoardInfoAsync ^CBSuFFtq

Await ^2DuE1arC

CONDITIONDATA
        (byte) ^g5wl2PHy

Get by passing the 
selected PCB's 
ConditionGUID ^APLGAxt9

Parse (convert to readable data) using ^lMd6Wv6S

Pass selected board object ^fHvlBYFA

Create BoardInfo = Selected Board + CONDITIONDATA ^dfgc8ZM4

BoardInfo ^M6Ep6cSp

inspResultUpdater ^dw1tHtsj

Invoke ^FIwzE9F9

Main ViewModel ^hFi6H9bo

UpdatedResultData ^BjQoVjeh

Handle ^nbjSAKtd

Pass to ^EV7yb54Z

Current ViewModel ^i8zg4c3f

UpdateBoard ^EO2MMyPS

Get BoardInfo as para ^fjaG0n5a

Send data to other Controls ^xK8JcNho

CreateInspectionItemsAsync ^Kz9bbQno

Await ^689N5PR5

return ^Ht1pDiLG

inspitems ^l0RfK9i3

pass to ^iykRfxGE

Inspitems ^JATfgcgq

DbConnect ViewModel ^dYAvdLCe

boardsInfos ^SyaoMunX

Add to ^MfnyA1Cp

Add to ^RMVeK9Cq

[boardsInfos is a property in DbConnect ViewModel
This property save all data that has been created so if user 
selected it again, the program don't have to create it again] ^QN6MnzzG

1 ^pWxYmWhm

2 ^KFsZ49sS

Invoke ^gNLc0dIP

boardInfo ^KSzSVCIX

Get from boardInfos ^YQ2KhAqR

Pass to ^WqMHxdPF

inspItems ^MfpMs5tK

Get from boardInfos ^5uLFWYVM

RaisePropertyChanged ^JXgr1Z3e

1.1 ^Km8ZPwgs

1.2 ^iZfu1YB9

1 ^c7hsFn1K

2 ^rWdBpeLm

1.1 ^1rSqbWm0

1.2 ^vqlbYsk3

Invoke and pass null ^KCSYHi8C

POPUlATE INSPECTIONITEMS AND ARRAYS DATA GRID ^Jyh0ilGE

Selected PCBItem changed ^oYuRHNma

[Get condition data only] ^owJcVV4a

[CreateBoardInfoAsync create 
data for array data grid] ^4T40NjuQ

[It also need condition data in 
byte format for board size info] ^uYhWzbWO

[This func is in CCIResultParser] ^UmTRpWra

[This func is in ResultDbReader] ^quoYd7cs

ResultDBReader ^oRS1TJuo

CreateInspectionItemsAsync ^LODHKiMf

Creates ^F6il80c9

Contains ^3tgX9r7t

Keys of ^lVWIGlPs

[Get all overview tables needed from specify ResultDBName] ^4UpwUazF

Uses to create InspItems ^qg0ciWZ2

Each one of this get ORM from DB then convert to DTO
    + ORM data type is the item table name
    + DTO data type is InspectionItem ^qu7nTP8h

Add to a list of InpsItems here ^HCsguqsc

Z ^npWbc3c4

Y ^tOi2r79E

X ^jOBlFePc

X ^BGMSNTTT

Y ^0wHshjOE

Z ^dPFZRATQ

[Handle when PCBItems changed or when InspectionItemResults changed] ^VYORq4dF

ResultDBReader ^L7edmt0c

CCIResultParser ^eX5Bni69

ResultProvider ^ZYb0QUD0

Help create data needed on the left data grids:
    + PCBItems
    + InspectionItems
    + InspectionItemResults ^yNRwP3su

Help create data needed on the left data grids:
    + Arrays
    + Components
    + ComponentDrawInfos
    + InspectionProperties ^gaf0PtvF

Convert IMeasureValue inside an InspectionItemResult
to corresponding defect data.
Functions in this class usually get called when the control
need data related to multiple different grids. ^IgwFQYe1

POPULATE OTHER DATA GRIDS ^wJ1vQ4rm

POPULATE INSPECTIONITEMRESULTS ^CfPR3FQk

POPULATE COMPONENTS, COMPONENT DRAW INFOS, INSPECTION PROPERTIES ^6ZasjKB3

Each one of this get ORM from DB then convert to DTO
    + ORM data type is the item result table name
    + DTO data type is InspectionItemResult ^mymuNVO7

Called these in their body ^wvnqofMR

Use these ^K3WTSsew

to create this ^q0pvSMMw

Sample flow ^AsyHPcks

General Informtation ^qn7yU4p5

Coating defect : Insufficient ^bfe7QFT7

Coating defect : Bubble ^c2czdslW

Coating defect : Crack ^lOsWTH5q

NonCoating defect : Splash ^hXWt6DGk

COATING + NONCOATING ^vsgfsqbK

THICKNESS ^yuh1kGhR

IC ^6qT36RYS

OVERCOATING ^DMGEJCY2

THICKNESS EX(TENSION) ^ZKBekOyX

UNDERFILL ^t3MCXuA5

{ ^TuczX1HX

Expanded ^KRd87zSx

Called these in their body ^uTyeZVYt

LIST ^rwZ2mnxZ

Get data from TB_CCI_XXX_OVERVIEW ^ZCsWt4MC

Get data from TB_CCI_XXX_<Corresponding Measure Item> ^q4VuAbd0

Get data from KY_AOI.dbo.TB_AOIPCB ^aSOyQ6Py

LIST ^UbHAlWdS

LIST ^89pnR2Ww

PCBITEMS ^VzPaWlpP

Data in left grid ^jjiujZ5v

Data in right grid ^1RQBrxdZ

Component changed so this will happen ^V2X4t0yI

[Component Draw Info will also be updated] ^szNXAlDW

Component changed so this will also happen ^XStCAzAH

Call this ^b51NsIRG

Call this ^bscfRPX9

Current ViewModel ^J5zKJFfp

UpdateItemResult ^pPrpn90n

Get SelectedInspItem as para ^2xDYvaXW

Send data to other Controls ^qLzWV5Yt

Sample ^cXKkKCeQ

Get SelectedInspItemResult as para ^NGVIX0mJ


# Embedded files
771b22b3bf824bfa1ad5f787570f3b7b67cedb85: [[Pasted Image 20240129135659_911.png]]
f08f5a5c87833d6a9fa69608611c3c5003529928: [[Pasted Image 20240129140029_928.png]]
b4466cafc2bf134185496b55ddad3ffd70234e7d: [[Pasted Image 20240205094425_015.png]]
d52a2be1748d859109575005233de3def646150b: [[Pasted Image 20240205094752_003.png]]
678041dfecf7ecc698e3952b662cc495f47b8d5f: [[Pasted Image 20240129141021_391.png]]
21c6d8b9894d2dd175869cc5cef70627201fd5ae: [[Pasted Image 20240129141523_438.png]]
30131359b4d23d486e0dcb42eaf8b504c876e27e: [[Pasted Image 20240129142458_197.png]]
73584240d8d994d39149a886c54857279059c084: [[Pasted Image 20240129144433_990.png]]
5251fddcb9b586300f9a22dfec78a449b182e494: [[Pasted Image 20240129145148_752.png]]
b55993223943985fab03f8d737dd5d468c0a8a0e: [[Pasted Image 20240129154328_992.png]]
36abd36d2c8c2556ba4b2f0896ef6671032bde84: [[Pasted Image 20240129155139_993.png]]
da7f1d7614cd1ee17cb8c6e4c70bf1be69a7ad21: [[Pasted Image 20240129161336_994.png]]
e6eef5f2d9ebefcbc9e6e9d857f895d9290437a7: [[Pasted Image 20240205100936_498.png]]
99546c1915d73087b33992c90dc4cdc16716f674: [[Pasted Image 20240202133137_100.png]]
c10555ce38240187e07582a50a935848cb22309e: [[Pasted Image 20240202133223_108.png]]
b098342fd39a640720d08ead3fd39295bd7c34ce: [[Pasted Image 20240202135353_995.png]]
71cbfdb3dffabfcc528868b84c0e6ad55409afbf: [[Pasted Image 20240202140745_248.png]]
c0f232272baf8178f58963be283c03071cb11777: [[Pasted Image 20240202140851_267.png]]
be07c77f5fb5d9a74157baf53abab76d7db8f3f2: [[Pasted Image 20240202142727_976.png]]
3e4c612bba5d3e49da99528062a9623fb76293aa: [[Pasted Image 20240202131340_080.png]]
02599db85914c06fc5008454440cd927947300af: [[Pasted Image 20240201161235_596.png]]
ec98425227d056bee29653ac021dfccfee948d0a: [[Pasted Image 20240205135720_587.png]]
99aee55d41b75c790cd894616689044b3f29c968: [[Pasted Image 20240201163818_683.png]]
5a37291430c63e6e34b0bcb73e4bcda3b6548290: [[Pasted Image 20240202081844_991.png]]
d535e0549394692c4be4a077fc1da946e7e87cd4: [[Pasted Image 20240202081930_990.png]]
2805bbe0fc2d5da322022f58df9e1fe55179e931: [[Pasted Image 20240131145023_047.png]]
b89d1e5a681c16226e822d099adccea9943c46ee: [[Pasted Image 20240202082621_996.png]]
475dbd3cde8352927e38726f39e6ae2066b35beb: [[Pasted Image 20240202082819_984.png]]
3896f173e8bc9132ae468b06792e641bfd627386: [[Pasted Image 20240202082904_989.png]]
e9b42ef1853a44baaa8fdd0170cc54ca3772b8b1: [[Pasted Image 20240202082935_991.png]]
4d96324e56095fdb25f0bbede3ea0b613ac33c65: [[Pasted Image 20240202091642_050.png]]
c10d5bd095f84727de3818e9a5ed803366ea0080: [[Pasted Image 20240202091824_304.png]]
75162803afca86f09621116dc4e258dc6e00d06c: [[Pasted Image 20240202091839_304.png]]
feb0e0e370f6f0ecbfb212196a2a14f8c6f23f16: [[Pasted Image 20240202092240_323.png]]
d41e625d1af52bd2eb05a93fd3982e4bcc2a1bd9: [[Pasted Image 20240202092325_325.png]]
d81c58926674be9d2fde46f320a37e9c7d0a5ec5: [[Pasted Image 20240202094109_016.png]]
aee86f6268e97a5d8266bac0f9cc2badbd0fb29b: [[Pasted Image 20240202094109_028.png]]
42d6401c49f2ca80cbdf420805a3c0e7cd229298: [[Pasted Image 20240202094124_017.png]]
b77b896361e32da5af61dd4599795148722690d4: [[Pasted Image 20240202094154_018.png]]
916dba59f86261d638efc6e5f392439369363492: [[Pasted Image 20240202094209_019.png]]
e9226c55d4fb6744cd136668eaec647747b687e4: [[Pasted Image 20240202094224_021.png]]
00cfb228d3ef30068eb1b4c878cf91f26867342c: [[Pasted Image 20240202094239_022.png]]
dfc889e436978d1ac02bea99cd0658b5c157e168: [[Pasted Image 20240202094325_029.png]]
b4df204c0941095dde0093b1dd0a1bb2cb7345c8: [[Pasted Image 20240202094749_073.png]]
390366ff42810ea78c404a581ffb369a3ab648e1: [[Pasted Image 20240202094804_075.png]]
97820b633bef1f91f2088efca540b6901c944ab6: [[Pasted Image 20240202094819_076.png]]
5cb822b6f6e1db0beaf623f6392f1e2ffad090e0: [[Pasted Image 20240202094819_079.png]]
0904ce05555e407d2d7759025500d4af5be47bf7: [[Pasted Image 20240202095119_089.png]]
504b8b86bf6c7f7cf1b27a164d00f774c3bdb7ce: [[Pasted Image 20240202095204_091.png]]
182dcdaacf5e1b7ca6f0479378771c91c7dcfea9: [[Pasted Image 20240202095219_092.png]]
20c28d0389488be998968f02c9cb10423d0c6b27: [[Pasted Image 20240202095234_093.png]]
f129603ccb702a7eab889284bdfb5ab9794301ea: [[Pasted Image 20240202095924_998.png]]
ba4f9196028c24bbc8289ca2b2392f14824502e5: [[Pasted Image 20240202095925_003.png]]
d87c19820158696f3fd1a6d457b0ec4018efdc8a: [[Pasted Image 20240202095939_999.png]]
a19825e6fb295b56b1bd15324fa4ff60aee08cf4: [[Pasted Image 20240202095940_003.png]]
e588f68bb8da6944042672e93135ea45ed163458: [[Pasted Image 20240202095955_000.png]]
572f62bed669777905468c7d74fa7d85c752749a: [[Pasted Image 20240202100010_001.png]]
dcca27879461f85170ff5c1eab8d63846d8b1e06: [[Pasted Image 20240202100010_005.png]]
c469d8ee22d2ee5c8405ca5164ac8276ea3db75f: [[Pasted Image 20240202100302_040.png]]
9625d14fa0354f9fbe7b6dde82fdc13a0369c8d2: [[Pasted Image 20240202100332_041.png]]
6d7b44461b641ec1387da8477e799e00d6424f7e: [[Pasted Image 20240202100403_048.png]]
0a26c11a7ed0de37550ff2435ee87a94f3bc89ef: [[Pasted Image 20240202100434_053.png]]
ade9f5336923b8644c1ec071e3d7d62aa68fa9e3: [[Pasted Image 20240202100449_053.png]]
f0885f2cb273122a0105ac438048cdff4f6913e5: [[Pasted Image 20240202100619_060.png]]
c507b88f496c7f0c65ce062b0656d9e166267dd5: [[Pasted Image 20240202100649_064.png]]
02370b1cec1018509df6b71607b61226b7cd9130: [[Pasted Image 20240205140125_680.png]]
553111c6c118fd0bd573b70bb50dd7aa92d41357: [[Pasted Image 20240205140213_691.png]]
8da848eb342a91855d51ca815d190c836711706a: [[Pasted Image 20240205140809_750.png]]
df9b1e8a0bd15cc9dcd899699d219c66c9fb12a9: [[Pasted Image 20240205140926_756.png]]
59757008b4d34cd7ed3a1f1baed5a5ddc33a369f: [[Pasted Image 20240129135133_008.png]]
45e636a8707ed34f6cf2bf17c0b76a0195112116: [[Pasted Image 20240202084812_495.png]]
626a6128e4781d84a26dd8bab91d43123874eecd: [[Pasted Image 20240201162314_452.png]]
94f601f9bb111eafe30aed90f5e69a65415e0cfe: [[Pasted Image 20240201162348_460.png]]
a31d109a60f030cbcc60ddf2e1a11037709a604c: [[Pasted Image 20240201162434_463.png]]
70258370012729a3c0a4befd76ab0dfaaa140b7b: [[Pasted Image 20240201162550_477.png]]
bf04029209fa74d56bda23add31154fce1ddb84d: [[Pasted Image 20240205142152_728.png]]
f17275bb126c3e9ecf4d0dd65f5eda4ee3a62c42: [[Pasted Image 20240205142222_731.png]]
5e76ba11cff7bc56d5f8b5f4e4994a65a063f560: [[Pasted Image 20240205142238_733.png]]
300f320d015d53a25ba5e31e434e9f9b0680bb09: [[Pasted Image 20240205142523_991.png]]
9429cb2605d1565b462e589b8978ed64e751e46e: [[Pasted Image 20240205142615_001.png]]

%%
# Drawing
```json
{
	"type": "excalidraw",
	"version": 2,
	"source": "https://github.com/zsviczian/obsidian-excalidraw-plugin/releases/tag/2.0.18",
	"elements": [
		{
			"type": "image",
			"version": 134,
			"versionNonce": 1009129349,
			"isDeleted": false,
			"id": "OjkEhpOrvimlCPOIF0y9u",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8727.111789720184,
			"y": 2385.0336007243473,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 2092.8,
			"height": 545,
			"seed": 1014774183,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "UBow9MxWuWjvmv6487H78",
					"type": "arrow"
				},
				{
					"id": "dbGqU_Yb1JenhTTd--jeC",
					"type": "arrow"
				},
				{
					"id": "n-f9iWo9xH1a5i4kvoo_E",
					"type": "arrow"
				}
			],
			"updated": 1712906619438,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "df9b1e8a0bd15cc9dcd899699d219c66c9fb12a9",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "text",
			"version": 137,
			"versionNonce": 1345645063,
			"isDeleted": false,
			"id": "ABuNEKf8",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1400.5547771194945,
			"y": -692.8549895883322,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 90.21994018554688,
			"height": 25,
			"seed": 917859335,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726348,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "Main View",
			"rawText": "Main View",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Main View",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "rectangle",
			"version": 306,
			"versionNonce": 1155718885,
			"isDeleted": false,
			"id": "8HcdMzSV8Hp0Vpg9gXw2j",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1413.888151142932,
			"y": -705.3549895883322,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 261.6666259765625,
			"height": 252.5,
			"seed": 1633178407,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1712906619438,
			"link": null,
			"locked": false
		},
		{
			"type": "rectangle",
			"version": 187,
			"versionNonce": 376090091,
			"isDeleted": false,
			"id": "jbTKDV8hZuq_4v0MmjUbf",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1383.0547771194945,
			"y": -645.3528067176298,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 210.83331298828125,
			"height": 178.33331298828125,
			"seed": 22107719,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1712906619438,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 170,
			"versionNonce": 694935017,
			"isDeleted": false,
			"id": "esLuTPxE",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1363.888151142932,
			"y": -631.1861502234892,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 148.51988220214844,
			"height": 25,
			"seed": 1031599463,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726349,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "DbConnect View",
			"rawText": "DbConnect View",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "DbConnect View",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 147,
			"versionNonce": 1513623847,
			"isDeleted": false,
			"id": "MKd157BM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1344.721403096057,
			"y": -553.6883025766134,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 140.9998779296875,
			"height": 25,
			"seed": 757575815,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726349,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "connectButton",
			"rawText": "connectButton",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "connectButton",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "rectangle",
			"version": 154,
			"versionNonce": 1045047717,
			"isDeleted": false,
			"id": "wsZBxb0VZflfwvj0m2eiX",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1350.5547160843382,
			"y": -555.3549895883322,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 154.16668701171875,
			"height": 32.5,
			"seed": 1242433447,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "1_2WuQRFBszAQWitRrUDG",
					"type": "arrow"
				}
			],
			"updated": 1712906619438,
			"link": null,
			"locked": false
		},
		{
			"type": "arrow",
			"version": 133,
			"versionNonce": 1107455787,
			"isDeleted": false,
			"id": "1_2WuQRFBszAQWitRrUDG",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1192.221403096057,
			"y": -538.6883636117697,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 225,
			"height": 0.83331298828125,
			"seed": 576486087,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619438,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "wsZBxb0VZflfwvj0m2eiX",
				"focus": 0.0069975266662895795,
				"gap": 4.1666259765625
			},
			"endBinding": {
				"elementId": "Hx8C0gUP",
				"focus": -0.091903750969304,
				"gap": 13.3333740234375
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					225,
					0.83331298828125
				]
			]
		},
		{
			"type": "text",
			"version": 171,
			"versionNonce": 300562633,
			"isDeleted": false,
			"id": "Hx8C0gUP",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -953.8880290726195,
			"y": -551.1883330941915,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 158.25987243652344,
			"height": 25,
			"seed": 1253018087,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "1_2WuQRFBszAQWitRrUDG",
					"type": "arrow"
				},
				{
					"id": "w57wo-IvLPgOz22b3acdl",
					"type": "arrow"
				}
			],
			"updated": 1715236726349,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "ConnectCommand",
			"rawText": "ConnectCommand",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "ConnectCommand",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 186,
			"versionNonce": 702632395,
			"isDeleted": false,
			"id": "w57wo-IvLPgOz22b3acdl",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -780.554655049182,
			"y": -537.8550506234884,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 231.45835876464844,
			"height": 0,
			"seed": 269726983,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619438,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Hx8C0gUP",
				"focus": 0.06666259765625,
				"gap": 15.073501586914062
			},
			"endBinding": {
				"elementId": "d1iK0Clq",
				"focus": 0.0533349609375,
				"gap": 12.078330993652344
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					231.45835876464844,
					0
				]
			]
		},
		{
			"type": "text",
			"version": 137,
			"versionNonce": 1559086181,
			"isDeleted": false,
			"id": "gl4bzUh4",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1116.3880290726195,
			"y": -566.1883636117697,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 62.265625,
			"height": 23,
			"seed": 1778744359,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619438,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Bind to",
			"rawText": "Bind to",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Bind to",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 182,
			"versionNonce": 84598891,
			"isDeleted": false,
			"id": "Mdsr5x7C",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -703.3541545609007,
			"y": -566.8550506234884,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 72.265625,
			"height": 23,
			"seed": 745627463,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619438,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Execute",
			"rawText": "Execute",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Execute",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 144,
			"versionNonce": 1306652613,
			"isDeleted": false,
			"id": "3TuugcSI",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -695.554655049182,
			"y": -527.0217376352072,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 54.462890625,
			"height": 23,
			"seed": 2041831015,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619438,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Async",
			"rawText": "Async",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Async",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 223,
			"versionNonce": 59331655,
			"isDeleted": false,
			"id": "d1iK0Clq",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -537.0179652908812,
			"y": -549.6883636117697,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 139.11988830566406,
			"height": 25,
			"seed": 1956215175,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "w57wo-IvLPgOz22b3acdl",
					"type": "arrow"
				},
				{
					"id": "CnJhvT_qQiS6a6QtWf-Y1",
					"type": "arrow"
				}
			],
			"updated": 1715236726351,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "TryConnectDB",
			"rawText": "TryConnectDB",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "TryConnectDB",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "image",
			"version": 306,
			"versionNonce": 435793701,
			"isDeleted": false,
			"id": "2_BY60b98szslXvHVPA7T",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -726.7808066028799,
			"y": -450.3115584216398,
			"strokeColor": "transparent",
			"backgroundColor": "#ffc9c9",
			"width": 514.744018298878,
			"height": 519.517533151727,
			"seed": 845066407,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619438,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "771b22b3bf824bfa1ad5f787570f3b7b67cedb85",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 159,
			"versionNonce": 1595408811,
			"isDeleted": false,
			"id": "mtRNq0ln2vCi3ILcc45FJ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -720.1758637761013,
			"y": 88.76645809419153,
			"strokeColor": "transparent",
			"backgroundColor": "#ffc9c9",
			"width": 447,
			"height": 279,
			"seed": 1823474631,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619438,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "f08f5a5c87833d6a9fa69608611c3c5003529928",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "text",
			"version": 431,
			"versionNonce": 1067614853,
			"isDeleted": false,
			"id": "wPFDQQFl",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1063.6059346197148,
			"y": 195.08664156373004,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffc9c9",
			"width": 305.732421875,
			"height": 69,
			"seed": 1940517607,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619438,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "[The Showindicator function helps \nshow loading screen\nwhile waiting to connect to DB]",
			"rawText": "[The Showindicator function helps \nshow loading screen\nwhile waiting to connect to DB]",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "[The Showindicator function helps \nshow loading screen\nwhile waiting to connect to DB]",
			"lineHeight": 1.15,
			"baseline": 64
		},
		{
			"type": "text",
			"version": 325,
			"versionNonce": 1671468107,
			"isDeleted": false,
			"id": "IYLGzjfM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1139.508989359998,
			"y": -266.4232784180678,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffc9c9",
			"width": 402.44140625,
			"height": 46,
			"seed": 1608056327,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619438,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "[ConnectionTest is a function implemented in \nResultDbReader class]",
			"rawText": "[ConnectionTest is a function implemented in \nResultDbReader class]",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "[ConnectionTest is a function implemented in \nResultDbReader class]",
			"lineHeight": 1.15,
			"baseline": 41
		},
		{
			"type": "rectangle",
			"version": 234,
			"versionNonce": 328853989,
			"isDeleted": false,
			"id": "pUg0L16xuJjGtoC3rVqa8",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1028.8958317683284,
			"y": -645.3528067176298,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 642.083465576172,
			"height": 178.33331298828125,
			"seed": 129981033,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1712906619438,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 192,
			"versionNonce": 1402615721,
			"isDeleted": false,
			"id": "fQNABkZF",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1009.7292057917659,
			"y": -631.1861502234892,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 148.51988220214844,
			"height": 25,
			"seed": 1100621129,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726352,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "DbConnect View",
			"rawText": "DbConnect View",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "DbConnect View",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 239,
			"versionNonce": 1995269445,
			"isDeleted": false,
			"id": "CnJhvT_qQiS6a6QtWf-Y1",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -391.60448044996906,
			"y": -537.5428857032584,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 436.8369986794212,
			"height": 2.4668792444283554,
			"seed": 1981455399,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619438,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "d1iK0Clq",
				"focus": -0.0435784570453937,
				"gap": 6.293596535248071
			},
			"endBinding": {
				"elementId": "l4gBP0UU",
				"focus": -0.15446251918824216,
				"gap": 8.981773376464844
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					436.8369986794212,
					2.4668792444283554
				]
			]
		},
		{
			"type": "text",
			"version": 323,
			"versionNonce": 1946497895,
			"isDeleted": false,
			"id": "l4gBP0UU",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 54.21429160591697,
			"y": -549.1005780705605,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 148.8998565673828,
			"height": 25,
			"seed": 1501264329,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "CnJhvT_qQiS6a6QtWf-Y1",
					"type": "arrow"
				}
			],
			"updated": 1715236726352,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "ConnectionTest",
			"rawText": "ConnectionTest",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "ConnectionTest",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "image",
			"version": 248,
			"versionNonce": 1291482277,
			"isDeleted": false,
			"id": "vLasGwbt2ZTUy0HOpTqtO",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -182.89601996006024,
			"y": -498.0450335352415,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 608,
			"height": 54,
			"seed": 1211399719,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619438,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "b4466cafc2bf134185496b55ddad3ffd70234e7d",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 141,
			"versionNonce": 1838175275,
			"isDeleted": false,
			"id": "gV-OLKfjIaJ-1SptfXlvD",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -212.99847500013743,
			"y": -896.735278796165,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 685,
			"height": 261,
			"seed": 1606893577,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619438,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "d52a2be1748d859109575005233de3def646150b",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "text",
			"version": 188,
			"versionNonce": 829331461,
			"isDeleted": false,
			"id": "MLHxtL5P",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -64.72064296888743,
			"y": -624.013110827415,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 350.5078125,
			"height": 23,
			"seed": 784826697,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619438,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "[Try connect to Db with this information]",
			"rawText": "[Try connect to Db with this information]",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "[Try connect to Db with this information]",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "rectangle",
			"version": 257,
			"versionNonce": 1416271563,
			"isDeleted": false,
			"id": "jIznicfjGGenagmjHPFYF",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1439.52523894121,
			"y": -936.0690471107793,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 1935.7141113281248,
			"height": 1338.095179966518,
			"seed": 674509959,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1712906619438,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 173,
			"versionNonce": 547838601,
			"isDeleted": false,
			"id": "w76fXodz",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1401.6133924868827,
			"y": -897.7211149093064,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 317.4118957519531,
			"height": 45,
			"seed": 1841414249,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726352,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "CONNECT TO DB",
			"rawText": "CONNECT TO DB",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "CONNECT TO DB",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "rectangle",
			"version": 359,
			"versionNonce": 2015593835,
			"isDeleted": false,
			"id": "StcgjNHPKexrN_xlVEbcE",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 709.7396104807437,
			"y": -936.0690471107793,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 4015.7141113281236,
			"height": 1031.428676060268,
			"seed": 1963078633,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1712906619438,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 314,
			"versionNonce": 603467399,
			"isDeleted": false,
			"id": "A2nviXog",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 747.6514569350709,
			"y": -897.7211149093064,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 652.0318603515625,
			"height": 45,
			"seed": 71683785,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726353,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "POPULATE PCBITEMS DATA GRID",
			"rawText": "POPULATE PCBITEMS DATA GRID",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "POPULATE PCBITEMS DATA GRID",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "text",
			"version": 323,
			"versionNonce": 1506615657,
			"isDeleted": false,
			"id": "NGy853j9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 765.0996837543389,
			"y": -683.9636505605619,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 90.21994018554688,
			"height": 25,
			"seed": 410210087,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726353,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "Main View",
			"rawText": "Main View",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Main View",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "rectangle",
			"version": 527,
			"versionNonce": 78412325,
			"isDeleted": false,
			"id": "seOygpS7pK5K1v1zWlPSZ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 751.7663097309014,
			"y": -696.4636505605619,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 320.833251953125,
			"height": 255,
			"seed": 330093127,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false
		},
		{
			"type": "rectangle",
			"version": 389,
			"versionNonce": 1175152299,
			"isDeleted": false,
			"id": "LuwY9X0qoWWPGRNLD7I1G",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 782.5996837543389,
			"y": -639.7969635488432,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 254.99993896484378,
			"height": 178.33331298828125,
			"seed": 1862857063,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 350,
			"versionNonce": 1310639527,
			"isDeleted": false,
			"id": "5qSo3opz",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 801.7663097309014,
			"y": -625.6303070547026,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 100.67991638183594,
			"height": 25,
			"seed": 1137978503,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726353,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "DbConnect",
			"rawText": "DbConnect",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "DbConnect",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 375,
			"versionNonce": 1012337737,
			"isDeleted": false,
			"id": "7qhMRLVE",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 820.9330577777764,
			"y": -544.7969635488432,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 173.47982788085938,
			"height": 25,
			"seed": 1305150375,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726353,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "pcbRefreshButton",
			"rawText": "pcbRefreshButton",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "pcbRefreshButton",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "rectangle",
			"version": 357,
			"versionNonce": 1112929509,
			"isDeleted": false,
			"id": "ChIEULEoazm_0WbBIgAsI",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 815.0997447894952,
			"y": -545.6303070547026,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 186.66668701171875,
			"height": 31.666656494140625,
			"seed": 488240839,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "sScRa8j_k8bdCGA4NZW6e",
					"type": "arrow"
				}
			],
			"updated": 1712906619439,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 351,
			"versionNonce": 564492487,
			"isDeleted": false,
			"id": "pSN7eGEw",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1223.635230218208,
			"y": -540.3500271321649,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 231.39979553222656,
			"height": 25,
			"seed": 140346855,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "sScRa8j_k8bdCGA4NZW6e",
					"type": "arrow"
				},
				{
					"id": "iyoObG6CPBNRw2ocQH7on",
					"type": "arrow"
				}
			],
			"updated": 1715236726354,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "PcbGridRefreshCommand",
			"rawText": "PcbGridRefreshCommand",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "PcbGridRefreshCommand",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 711,
			"versionNonce": 670044229,
			"isDeleted": false,
			"id": "sScRa8j_k8bdCGA4NZW6e",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1002.7664318012139,
			"y": -532.3802855974275,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 214.33100919003203,
			"height": 2.3373287097545443,
			"seed": 1587687687,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "ChIEULEoazm_0WbBIgAsI",
				"focus": -0.21434937978684676,
				"gap": 1
			},
			"endBinding": {
				"elementId": "pSN7eGEw",
				"focus": 0.0624848445958404,
				"gap": 6.537789226962104
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					214.33100919003203,
					2.3373287097545443
				]
			]
		},
		{
			"type": "text",
			"version": 298,
			"versionNonce": 453755531,
			"isDeleted": false,
			"id": "HxCW8nX9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1103.2334094108799,
			"y": -562.1660344603862,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 62.265625,
			"height": 23,
			"seed": 1499756583,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Bind to",
			"rawText": "Bind to",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Bind to",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 870,
			"versionNonce": 1047510949,
			"isDeleted": false,
			"id": "iyoObG6CPBNRw2ocQH7on",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1469.5966714492538,
			"y": -529.1517865629633,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 232.49999999999994,
			"height": 0,
			"seed": 1318116167,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "pSN7eGEw",
				"focus": -0.1041407544638696,
				"gap": 14.56164569881912
			},
			"endBinding": {
				"elementId": "IWwIbnzG",
				"focus": -0.011102329863342674,
				"gap": 11.036689758300781
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					232.49999999999994,
					0
				]
			]
		},
		{
			"type": "text",
			"version": 435,
			"versionNonce": 1898527019,
			"isDeleted": false,
			"id": "isflkmm6",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1546.797171937535,
			"y": -558.9572526979738,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 72.265625,
			"height": 23,
			"seed": 579066471,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Execute",
			"rawText": "Execute",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Execute",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 397,
			"versionNonce": 2080539397,
			"isDeleted": false,
			"id": "HMZbfz1W",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1554.5966714492538,
			"y": -519.1239397096925,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 54.462890625,
			"height": 23,
			"seed": 2047473031,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Async",
			"rawText": "Async",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Async",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 491,
			"versionNonce": 1344314153,
			"isDeleted": false,
			"id": "IWwIbnzG",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1713.1333612075546,
			"y": -541.790565686255,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 129.13987731933594,
			"height": 25,
			"seed": 744467623,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "iyoObG6CPBNRw2ocQH7on",
					"type": "arrow"
				},
				{
					"id": "LXzCHC8k0ieb_i4-NKOzi",
					"type": "arrow"
				}
			],
			"updated": 1715236726354,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "Showindicator",
			"rawText": "Showindicator",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Showindicator",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 570,
			"versionNonce": 875087847,
			"isDeleted": false,
			"id": "Kho0kBB1",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2092.673516307947,
			"y": -543.759005536205,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 128.87986755371094,
			"height": 25,
			"seed": 1966178247,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "LXzCHC8k0ieb_i4-NKOzi",
					"type": "arrow"
				},
				{
					"id": "s8BUheM6y0g2jlft6aWDM",
					"type": "arrow"
				}
			],
			"updated": 1715236726354,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "RefreshQuery",
			"rawText": "RefreshQuery",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "RefreshQuery",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 507,
			"versionNonce": 2089233003,
			"isDeleted": false,
			"id": "x2XXcTf2",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1936.667982198272,
			"y": -517.2495310944486,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 48.544921875,
			"height": 23,
			"seed": 1721495271,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Await",
			"rawText": "Await",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Await",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 914,
			"versionNonce": 1386694085,
			"isDeleted": false,
			"id": "LXzCHC8k0ieb_i4-NKOzi",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1851.6948551915398,
			"y": -529.650065006498,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 232.49999999999994,
			"height": 0,
			"seed": 455834119,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "IWwIbnzG",
				"focus": -0.028759945619440258,
				"gap": 9.421616664649264
			},
			"endBinding": {
				"elementId": "Kho0kBB1",
				"focus": -0.1287152423765565,
				"gap": 8.478661116407238
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					232.49999999999994,
					0
				]
			]
		},
		{
			"type": "image",
			"version": 372,
			"versionNonce": 44868875,
			"isDeleted": false,
			"id": "YDH2iBzyqR5kvE1Lm3ZZ1",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1932.6113035020087,
			"y": -467.9115468821766,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 439,
			"height": 238,
			"seed": 668770599,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "678041dfecf7ecc698e3952b662cc495f47b8d5f",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "freedraw",
			"version": 336,
			"versionNonce": 1201947941,
			"isDeleted": false,
			"id": "X4Am_kW2uerazGduWsvO3",
			"fillStyle": "hachure",
			"strokeWidth": 0.5,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1918.7779701686754,
			"y": -416.6892975332182,
			"strokeColor": "#e03131",
			"backgroundColor": "transparent",
			"width": 18.888834635416515,
			"height": 70,
			"seed": 1310450759,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0,
					1.111083984375
				],
				[
					0,
					7.777750651041629
				],
				[
					0,
					16.66666666666663
				],
				[
					0,
					21.111083984375
				],
				[
					0,
					24.444417317708258
				],
				[
					0,
					25.555501302083258
				],
				[
					0,
					27.77775065104163
				],
				[
					0,
					30
				],
				[
					-1.1111653645832575,
					32.22216796875
				],
				[
					-2.2222493489582575,
					33.33333333333326
				],
				[
					-3.3333333333332575,
					34.44441731770826
				],
				[
					-4.444498697916515,
					35.55550130208326
				],
				[
					-5.555582682291515,
					36.66666666666663
				],
				[
					-6.666666666666515,
					36.66666666666663
				],
				[
					-8.888916015625,
					36.66666666666663
				],
				[
					-11.111165364583258,
					36.66666666666663
				],
				[
					-12.222249348958258,
					36.66666666666663
				],
				[
					-13.333333333333258,
					35.55550130208326
				],
				[
					-15.555582682291515,
					33.33333333333326
				],
				[
					-15.555582682291515,
					32.22216796875
				],
				[
					-16.666666666666515,
					30
				],
				[
					-16.666666666666515,
					28.88883463541663
				],
				[
					-16.666666666666515,
					27.77775065104163
				],
				[
					-15.555582682291515,
					26.66666666666663
				],
				[
					-13.333333333333258,
					25.555501302083258
				],
				[
					-12.222249348958258,
					25.555501302083258
				],
				[
					-11.111165364583258,
					25.555501302083258
				],
				[
					-8.888916015625,
					25.555501302083258
				],
				[
					-6.666666666666515,
					27.77775065104163
				],
				[
					-4.444498697916515,
					31.111083984375
				],
				[
					-2.2222493489582575,
					33.33333333333326
				],
				[
					-1.1111653645832575,
					36.66666666666663
				],
				[
					0,
					38.88883463541663
				],
				[
					0,
					41.111083984375
				],
				[
					1.111083984375,
					44.44441731770826
				],
				[
					2.22216796875,
					46.66666666666663
				],
				[
					2.22216796875,
					50
				],
				[
					2.22216796875,
					52.22216796875
				],
				[
					2.22216796875,
					53.33333333333326
				],
				[
					2.22216796875,
					55.55550130208326
				],
				[
					2.22216796875,
					57.77775065104163
				],
				[
					1.111083984375,
					61.111083984375
				],
				[
					1.111083984375,
					63.33333333333326
				],
				[
					1.111083984375,
					64.44441731770826
				],
				[
					1.111083984375,
					66.66666666666663
				],
				[
					1.111083984375,
					67.77775065104163
				],
				[
					1.111083984375,
					68.88883463541663
				],
				[
					2.22216796875,
					70
				],
				[
					2.22216796875,
					70
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "text",
			"version": 331,
			"versionNonce": 663341995,
			"isDeleted": false,
			"id": "pW4f9idO",
			"fillStyle": "hachure",
			"strokeWidth": 0.5,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1784.6824427581266,
			"y": -392.92751926969584,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 108.5078125,
			"height": 18.4,
			"seed": 1841677159,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"fontSize": 16,
			"fontFamily": 2,
			"text": "[Clear old data]",
			"rawText": "[Clear old data]",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "[Clear old data]",
			"lineHeight": 1.15,
			"baseline": 14
		},
		{
			"type": "arrow",
			"version": 663,
			"versionNonce": 1200248965,
			"isDeleted": false,
			"id": "s8BUheM6y0g2jlft6aWDM",
			"fillStyle": "hachure",
			"strokeWidth": 0.5,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2228.664177150011,
			"y": -529.5139417891404,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 255.53510479193233,
			"height": 2.206256275841568,
			"seed": 925286023,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Kho0kBB1",
				"focus": 0.08634134881792514,
				"gap": 7.110793288352852
			},
			"endBinding": {
				"elementId": "0sU1cMtP",
				"focus": 0.11506288871396125,
				"gap": 14.266317568256454
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					255.53510479193233,
					2.206256275841568
				]
			]
		},
		{
			"type": "text",
			"version": 539,
			"versionNonce": 256450123,
			"isDeleted": false,
			"id": "pNQctWC8",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2319.430408402056,
			"y": -516.701195283827,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 48.544921875,
			"height": 23,
			"seed": 486233511,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Await",
			"rawText": "Await",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Await",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 343,
			"versionNonce": 362106377,
			"isDeleted": false,
			"id": "0sU1cMtP",
			"fillStyle": "hachure",
			"strokeWidth": 0.5,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2498.4655995101994,
			"y": -537.1152111367335,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 234.95977783203125,
			"height": 25,
			"seed": 108005575,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "s8BUheM6y0g2jlft6aWDM",
					"type": "arrow"
				},
				{
					"id": "GMNHjicK5sx9S7YFONSSl",
					"type": "arrow"
				}
			],
			"updated": 1715236726355,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "ExecuteSelectPCBQuery",
			"rawText": "ExecuteSelectPCBQuery",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "ExecuteSelectPCBQuery",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 751,
			"versionNonce": 1959075051,
			"isDeleted": false,
			"id": "GMNHjicK5sx9S7YFONSSl",
			"fillStyle": "hachure",
			"strokeWidth": 0.5,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2742.8860802053396,
			"y": -526.2767988409126,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 255.53510479193233,
			"height": 2.126487674763297,
			"seed": 1092704231,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "0sU1cMtP",
				"focus": -0.20166373662209358,
				"gap": 9.460702863108963
			},
			"endBinding": {
				"elementId": "eOKJlUtg",
				"focus": -0.2525240236956753,
				"gap": 10.258772929893894
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					255.53510479193233,
					2.126487674763297
				]
			]
		},
		{
			"type": "text",
			"version": 573,
			"versionNonce": 2069347141,
			"isDeleted": false,
			"id": "UCCDntDB",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2833.6523114573847,
			"y": -513.4414459418728,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 48.544921875,
			"height": 23,
			"seed": 857457415,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Await",
			"rawText": "Await",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Await",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 414,
			"versionNonce": 867513095,
			"isDeleted": false,
			"id": "eOKJlUtg",
			"fillStyle": "hachure",
			"strokeWidth": 0.5,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3008.6799579271656,
			"y": -538.8577443519522,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 277.71978759765625,
			"height": 25,
			"seed": 1289472551,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "GMNHjicK5sx9S7YFONSSl",
					"type": "arrow"
				},
				{
					"id": "KNZZQUZMnLHjDIKsnUyAh",
					"type": "arrow"
				}
			],
			"updated": 1715236726356,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "CreateInspectionBoardAsync",
			"rawText": "CreateInspectionBoardAsync",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "CreateInspectionBoardAsync",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 538,
			"versionNonce": 1034554021,
			"isDeleted": false,
			"id": "xEjhknWl",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3003.830515476317,
			"y": -493.530405946188,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "#ffc9c9",
			"width": 302.36328125,
			"height": 46,
			"seed": 994962759,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "[This is a function implemented in \nResultDbReader class]",
			"rawText": "[This is a function implemented in \nResultDbReader class]",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "[This is a function implemented in \nResultDbReader class]",
			"lineHeight": 1.15,
			"baseline": 41
		},
		{
			"type": "image",
			"version": 332,
			"versionNonce": 1372397099,
			"isDeleted": false,
			"id": "ekuTubK9ja__8vhX3PloI",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2516.7176527531024,
			"y": -420.7446829411655,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 1256.3589163505935,
			"height": 330.66668701171875,
			"seed": 1506716775,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "21c6d8b9894d2dd175869cc5cef70627201fd5ae",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "text",
			"version": 392,
			"versionNonce": 1404792325,
			"isDeleted": false,
			"id": "3gVEp3RH",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2793.89025145567,
			"y": -676.2565597286235,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 165.654296875,
			"height": 23,
			"seed": 1680508807,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "return List<Board>",
			"rawText": "return List<Board>",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "return List<Board>",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "freedraw",
			"version": 329,
			"versionNonce": 1786650827,
			"isDeleted": false,
			"id": "a_vnWJqUSdQE90ChFLNZb",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3145.973610220318,
			"y": -555.4232009639751,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 490.6249999999998,
			"height": 83.33332061767572,
			"seed": 1755184807,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					-1.041717529296875,
					0
				],
				[
					-2.083282470703125,
					-3.125
				],
				[
					-7.291717529296875,
					-26.04167938232422
				],
				[
					-14.583282470703125,
					-49.99999999999994
				],
				[
					-18.75,
					-66.66667938232416
				],
				[
					-20.833282470703125,
					-73.95832061767572
				],
				[
					-21.875,
					-79.16667938232416
				],
				[
					-22.916717529296875,
					-79.16667938232416
				],
				[
					-23.958282470703125,
					-79.16667938232416
				],
				[
					-30.208282470703125,
					-79.16667938232416
				],
				[
					-55.208282470703125,
					-79.16667938232416
				],
				[
					-103.125,
					-81.24999999999994
				],
				[
					-155.2082824707029,
					-83.33332061767572
				],
				[
					-197.91671752929665,
					-83.33332061767572
				],
				[
					-232.29171752929665,
					-83.33332061767572
				],
				[
					-258.3332824707029,
					-83.33332061767572
				],
				[
					-276.04164123535134,
					-83.33332061767572
				],
				[
					-295.8332824707029,
					-83.33332061767572
				],
				[
					-324.9999999999998,
					-82.29167938232416
				],
				[
					-344.79164123535134,
					-79.16667938232416
				],
				[
					-365.6249999999998,
					-74.99999999999994
				],
				[
					-380.2082824707029,
					-74.99999999999994
				],
				[
					-387.4999999999998,
					-73.95832061767572
				],
				[
					-396.8749999999998,
					-73.95832061767572
				],
				[
					-406.2499999999998,
					-73.95832061767572
				],
				[
					-418.7499999999998,
					-73.95832061767572
				],
				[
					-429.16664123535134,
					-73.95832061767572
				],
				[
					-437.4999999999998,
					-73.95832061767572
				],
				[
					-442.7082824707029,
					-73.95832061767572
				],
				[
					-446.8749999999998,
					-73.95832061767572
				],
				[
					-456.2499999999998,
					-73.95832061767572
				],
				[
					-460.41664123535134,
					-73.95832061767572
				],
				[
					-460.41664123535134,
					-72.91667938232416
				],
				[
					-460.41664123535134,
					-67.70832061767572
				],
				[
					-464.5832824707029,
					-54.16667938232416
				],
				[
					-465.6249999999998,
					-46.87499999999994
				],
				[
					-468.7499999999998,
					-37.5
				],
				[
					-472.91664123535134,
					-28.125
				],
				[
					-476.04164123535134,
					-20.83332061767578
				],
				[
					-479.16664123535134,
					-15.625
				],
				[
					-481.2499999999998,
					-14.583320617675781
				],
				[
					-481.2499999999998,
					-13.541679382324219
				],
				[
					-482.29164123535134,
					-11.458320617675781
				],
				[
					-484.3749999999998,
					-10.416679382324219
				],
				[
					-486.4582824707029,
					-8.333320617675781
				],
				[
					-489.5832824707029,
					-5.208320617675781
				],
				[
					-490.6249999999998,
					-5.208320617675781
				],
				[
					-490.6249999999998,
					-5.208320617675781
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "freedraw",
			"version": 298,
			"versionNonce": 478766437,
			"isDeleted": false,
			"id": "cgGOBm-14T6vVIGhwcs3W",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2653.2653277496147,
			"y": -582.5065215816509,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 37.5,
			"height": 33.33332061767578,
			"seed": 2044039623,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					1.0416412353515625,
					2.0833206176757812
				],
				[
					2.083282470703125,
					3.125
				],
				[
					2.083282470703125,
					5.208320617675781
				],
				[
					2.083282470703125,
					10.416641235351562
				],
				[
					-1.041717529296875,
					16.666641235351562
				],
				[
					-9.375,
					26.041641235351562
				],
				[
					-11.458358764648438,
					31.25
				],
				[
					-12.5,
					32.29164123535156
				],
				[
					-12.5,
					33.33332061767578
				],
				[
					-11.458358764648438,
					33.33332061767578
				],
				[
					-9.375,
					33.33332061767578
				],
				[
					-7.291717529296875,
					33.33332061767578
				],
				[
					-3.125,
					32.29164123535156
				],
				[
					3.125,
					31.25
				],
				[
					14.583282470703125,
					30.20832061767578
				],
				[
					20.833282470703125,
					30.20832061767578
				],
				[
					23.958282470703125,
					30.20832061767578
				],
				[
					25,
					30.20832061767578
				],
				[
					25,
					30.20832061767578
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "freedraw",
			"version": 307,
			"versionNonce": 1463144299,
			"isDeleted": false,
			"id": "z7LT0PuNq1x4Jk1jcx6T-",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2615.7653277496147,
			"y": -542.9231628170024,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 105.20828247070312,
			"height": 121.87499999999994,
			"seed": 552452327,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0,
					-1.0416793823242188
				],
				[
					0,
					-10.416679382324219
				],
				[
					-4.1666412353515625,
					-26.04167938232422
				],
				[
					-7.2916412353515625,
					-50
				],
				[
					-10.416641235351562,
					-79.16667938232416
				],
				[
					-11.458282470703125,
					-101.04167938232416
				],
				[
					-14.583282470703125,
					-109.37499999999994
				],
				[
					-17.708282470703125,
					-114.58335876464838
				],
				[
					-18.75,
					-114.58335876464838
				],
				[
					-19.791641235351562,
					-114.58335876464838
				],
				[
					-22.916641235351562,
					-115.62499999999994
				],
				[
					-34.375,
					-117.70835876464838
				],
				[
					-62.5,
					-121.87499999999994
				],
				[
					-80.20828247070312,
					-121.87499999999994
				],
				[
					-95.83328247070312,
					-121.87499999999994
				],
				[
					-102.08328247070312,
					-121.87499999999994
				],
				[
					-104.16664123535156,
					-121.87499999999994
				],
				[
					-105.20828247070312,
					-121.87499999999994
				],
				[
					-102.08328247070312,
					-101.04167938232416
				],
				[
					-102.08328247070312,
					-84.37499999999994
				],
				[
					-102.08328247070312,
					-70.83335876464838
				],
				[
					-102.08328247070312,
					-55.20835876464844
				],
				[
					-102.08328247070312,
					-41.66667938232422
				],
				[
					-102.08328247070312,
					-28.125
				],
				[
					-101.04164123535156,
					-21.875
				],
				[
					-101.04164123535156,
					-18.75
				],
				[
					-101.04164123535156,
					-16.66667938232422
				],
				[
					-101.04164123535156,
					-16.66667938232422
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "freedraw",
			"version": 294,
			"versionNonce": 996241605,
			"isDeleted": false,
			"id": "dXz7JJanMr1ACaorde--t",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2493.8903277496147,
			"y": -570.0065215816509,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 39.58335876464844,
			"height": 26.041641235351562,
			"seed": 259197959,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0,
					-1.0416412353515625
				],
				[
					3.125,
					0
				],
				[
					4.166717529296875,
					1.0416793823242188
				],
				[
					6.25,
					3.125
				],
				[
					9.375,
					7.291679382324219
				],
				[
					14.583358764648438,
					15.625
				],
				[
					18.75,
					21.875
				],
				[
					19.791717529296875,
					23.958358764648438
				],
				[
					21.875,
					25
				],
				[
					22.916717529296875,
					25
				],
				[
					25,
					21.875
				],
				[
					32.291717529296875,
					11.458358764648438
				],
				[
					37.5,
					1.0416793823242188
				],
				[
					39.58335876464844,
					0
				],
				[
					39.58335876464844,
					0
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "text",
			"version": 441,
			"versionNonce": 977185291,
			"isDeleted": false,
			"id": "q35YRT2q",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2392.5970338700326,
			"y": -728.2138787996283,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 302.32421875,
			"height": 46,
			"seed": 291631911,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "            Pass to PCBItems\n (which is source for pcbDataGrid)",
			"rawText": "            Pass to PCBItems\n (which is source for pcbDataGrid)",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "            Pass to PCBItems\n (which is source for pcbDataGrid)",
			"lineHeight": 1.15,
			"baseline": 41
		},
		{
			"type": "text",
			"version": 454,
			"versionNonce": 1940956393,
			"isDeleted": false,
			"id": "bXsqhlWl",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 4091.398094476106,
			"y": -541.5104312690705,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 186.41983032226562,
			"height": 25,
			"seed": 1059518023,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "KNZZQUZMnLHjDIKsnUyAh",
					"type": "arrow"
				},
				{
					"id": "-y4rCyoLEvVbrMxlWBRKE",
					"type": "arrow"
				}
			],
			"updated": 1715236726357,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "ExecuteQueryAsync",
			"rawText": "ExecuteQueryAsync",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "ExecuteQueryAsync",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 1201,
			"versionNonce": 143863979,
			"isDeleted": false,
			"id": "KNZZQUZMnLHjDIKsnUyAh",
			"fillStyle": "hachure",
			"strokeWidth": 0.5,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3295.829153932269,
			"y": -527.8104022834917,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 785.6474395228383,
			"height": 0.14800446397596545,
			"seed": 1872663911,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "eOKJlUtg",
				"focus": -0.11373976499537175,
				"gap": 9.429408407447227
			},
			"endBinding": {
				"elementId": "bXsqhlWl",
				"focus": -0.08249180552425356,
				"gap": 9.921501020998676
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					785.6474395228383,
					-0.14800446397596545
				]
			]
		},
		{
			"type": "text",
			"version": 719,
			"versionNonce": 1643496325,
			"isDeleted": false,
			"id": "mPaoEPt5",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3655.484382580147,
			"y": -506.7020924308539,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 48.544921875,
			"height": 23,
			"seed": 1779781767,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Await",
			"rawText": "Await",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Await",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "image",
			"version": 374,
			"versionNonce": 870179659,
			"isDeleted": false,
			"id": "a7SDOpOxX5MizL5aaAcTD",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3823.5381945290487,
			"y": -422.0112379168503,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 848.5229043889582,
			"height": 290.7778320312501,
			"seed": 1481090983,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "30131359b4d23d486e0dcb42eaf8b504c876e27e",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 683,
			"versionNonce": 1809648357,
			"isDeleted": false,
			"id": "-y4rCyoLEvVbrMxlWBRKE",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 4288.871527862382,
			"y": -529.4000318621628,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 166.66666666666652,
			"height": 1.1111246744791288,
			"seed": 2019695303,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "bXsqhlWl",
				"focus": -0.08266637853235728,
				"gap": 11.053603064010531
			},
			"endBinding": {
				"elementId": "CjHKtRSL",
				"focus": -0.08857601605673925,
				"gap": 10.000000000001819
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					166.66666666666652,
					1.1111246744791288
				]
			]
		},
		{
			"type": "text",
			"version": 318,
			"versionNonce": 1489451559,
			"isDeleted": false,
			"id": "CjHKtRSL",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 4465.53819452905,
			"y": -541.622240521017,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 68.19993591308594,
			"height": 25,
			"seed": 251734503,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "-y4rCyoLEvVbrMxlWBRKE",
					"type": "arrow"
				}
			],
			"updated": 1715236726357,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "Dapper",
			"rawText": "Dapper",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Dapper",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 284,
			"versionNonce": 2082760261,
			"isDeleted": false,
			"id": "pdSB4L3E",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 4338.871365101966,
			"y": -558.2889478777878,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 51.1328125,
			"height": 23,
			"seed": 422663431,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Using",
			"rawText": "Using",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Using",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 320,
			"versionNonce": 326093963,
			"isDeleted": false,
			"id": "pBarH0G1",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 4295.538194529048,
			"y": -671.622240521017,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 206.77734375,
			"height": 23,
			"seed": 1047469095,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "return IEnumarable<T>",
			"rawText": "return IEnumarable<T>",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "return IEnumarable<T>",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "freedraw",
			"version": 304,
			"versionNonce": 1974031781,
			"isDeleted": false,
			"id": "Q1Zl5VGwIW6QWcF8PvXf5",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 4491.093695831132,
			"y": -552.733324505392,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 231.11100260416652,
			"height": 75.55558268229163,
			"seed": 44498759,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0,
					-1.1111246744791288
				],
				[
					-2.22216796875,
					-11.111124674479129
				],
				[
					-5.55550130208303,
					-27.777791341145814
				],
				[
					-8.88883463541697,
					-43.33333333333326
				],
				[
					-11.11100260416697,
					-52.22224934895826
				],
				[
					-13.33333333333303,
					-58.88891601562494
				],
				[
					-13.33333333333303,
					-63.33333333333326
				],
				[
					-15.55550130208303,
					-65.55558268229163
				],
				[
					-17.77766927083303,
					-67.77779134114576
				],
				[
					-20,
					-69.99999999999994
				],
				[
					-20,
					-73.33333333333326
				],
				[
					-22.22216796875,
					-74.44445800781244
				],
				[
					-22.22216796875,
					-75.55558268229163
				],
				[
					-25.55550130208303,
					-75.55558268229163
				],
				[
					-34.4443359375,
					-75.55558268229163
				],
				[
					-63.33333333333303,
					-75.55558268229163
				],
				[
					-107.77766927083303,
					-75.55558268229163
				],
				[
					-153.33333333333348,
					-75.55558268229163
				],
				[
					-182.22216796875,
					-75.55558268229163
				],
				[
					-198.88883463541652,
					-75.55558268229163
				],
				[
					-201.11100260416652,
					-75.55558268229163
				],
				[
					-202.22216796875,
					-75.55558268229163
				],
				[
					-204.4443359375,
					-68.88891601562494
				],
				[
					-208.88883463541652,
					-57.77779134114576
				],
				[
					-217.77766927083303,
					-45.55558268229163
				],
				[
					-226.66666666666652,
					-32.222249348958314
				],
				[
					-230,
					-26.66666666666663
				],
				[
					-231.11100260416652,
					-25.55558268229163
				],
				[
					-231.11100260416652,
					-23.333333333333314
				],
				[
					-231.11100260416652,
					-23.333333333333314
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "freedraw",
			"version": 286,
			"versionNonce": 343671595,
			"isDeleted": false,
			"id": "V67kj-1TT7BkRexPwStSj",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 4259.982693226966,
			"y": -589.3999911720587,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 20,
			"height": 33.33333333333337,
			"seed": 943029863,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0,
					1.111083984375
				],
				[
					0,
					5.5555419921875
				],
				[
					-1.111165364583485,
					15.5555419921875
				],
				[
					-2.222330729166515,
					25.5555419921875
				],
				[
					-2.222330729166515,
					30
				],
				[
					-2.222330729166515,
					33.33333333333337
				],
				[
					-3.333333333333485,
					33.33333333333337
				],
				[
					0,
					30
				],
				[
					4.4443359375,
					23.333333333333314
				],
				[
					12.22216796875,
					11.111083984375
				],
				[
					16.666666666666515,
					6.666666666666686
				],
				[
					16.666666666666515,
					6.666666666666686
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "freedraw",
			"version": 309,
			"versionNonce": 60381445,
			"isDeleted": false,
			"id": "2RvBqzTYN4j-eRztQaOce",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 4222.204861195716,
			"y": -559.3999911720587,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 121.111083984375,
			"height": 84.44441731770826,
			"seed": 837228935,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0,
					-1.1111246744791856
				],
				[
					0,
					-10
				],
				[
					0,
					-18.888916015625
				],
				[
					0,
					-35.55558268229163
				],
				[
					0,
					-45.55558268229163
				],
				[
					-5.555501302083485,
					-63.333333333333314
				],
				[
					-8.888834635416515,
					-70
				],
				[
					-10,
					-71.11112467447913
				],
				[
					-11.111165364583485,
					-72.22224934895831
				],
				[
					-12.22216796875,
					-73.33333333333331
				],
				[
					-13.333333333333485,
					-75.55558268229163
				],
				[
					-14.444498697916515,
					-75.55558268229163
				],
				[
					-18.888834635416515,
					-76.66666666666663
				],
				[
					-31.111165364583485,
					-76.66666666666663
				],
				[
					-57.77783203125,
					-76.66666666666663
				],
				[
					-80,
					-76.66666666666663
				],
				[
					-93.33333333333348,
					-76.66666666666663
				],
				[
					-95.55550130208348,
					-76.66666666666663
				],
				[
					-96.66666666666652,
					-76.66666666666663
				],
				[
					-100,
					-76.66666666666663
				],
				[
					-107.77775065104152,
					-76.66666666666663
				],
				[
					-116.66666666666652,
					-76.66666666666663
				],
				[
					-117.77775065104152,
					-76.66666666666663
				],
				[
					-120,
					-76.66666666666663
				],
				[
					-121.111083984375,
					-76.66666666666663
				],
				[
					-121.111083984375,
					-75.55558268229163
				],
				[
					-120,
					-74.4444580078125
				],
				[
					-118.88883463541652,
					-67.77779134114581
				],
				[
					-115.55550130208348,
					-53.333333333333314
				],
				[
					-112.22216796875,
					-36.66666666666663
				],
				[
					-110,
					-23.333333333333314
				],
				[
					-108.88883463541652,
					-8.888916015625
				],
				[
					-107.77775065104152,
					3.3333333333333712
				],
				[
					-107.77775065104152,
					7.777750651041629
				],
				[
					-107.77775065104152,
					7.777750651041629
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "freedraw",
			"version": 286,
			"versionNonce": 537421259,
			"isDeleted": false,
			"id": "vRyMBdruXV10B4UtieMHx",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 4102.204861195716,
			"y": -553.8444491798712,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 28.888916015625,
			"height": 25.55558268229163,
			"seed": 888688807,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					1.111165364583485,
					0
				],
				[
					4.444498697916515,
					0
				],
				[
					6.666666666666515,
					2.2222086588541288
				],
				[
					8.888916015625,
					3.3333333333333712
				],
				[
					11.111165364583485,
					4.4444580078125
				],
				[
					14.444498697916515,
					6.666666666666629
				],
				[
					16.666666666666515,
					11.111124674479129
				],
				[
					17.77783203125,
					11.111124674479129
				],
				[
					20,
					6.666666666666629
				],
				[
					26.666666666666515,
					-8.888875325520814
				],
				[
					28.888916015625,
					-14.4444580078125
				],
				[
					28.888916015625,
					-14.4444580078125
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "text",
			"version": 387,
			"versionNonce": 259125349,
			"isDeleted": false,
			"id": "shkzeax4",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 4095.4830187477983,
			"y": -676.455533164246,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 124.501953125,
			"height": 23,
			"seed": 722838471,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "convert to List",
			"rawText": "convert to List",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "convert to List",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "freedraw",
			"version": 356,
			"versionNonce": 1294955627,
			"isDeleted": false,
			"id": "55nrg-7bFayNid142kn_l",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 4085.538357289466,
			"y": -554.955451784038,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 882.22216796875,
			"height": 91.11108398437494,
			"seed": 341545703,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					1.111002604166515,
					1.111083984375
				],
				[
					1.111002604166515,
					0
				],
				[
					-3.333333333333485,
					-6.666666666666629
				],
				[
					-15.5556640625,
					-24.4444580078125
				],
				[
					-26.66666666666697,
					-39.99999999999994
				],
				[
					-34.44449869791697,
					-49.99999999999994
				],
				[
					-44.44449869791697,
					-62.222249348958314
				],
				[
					-54.44449869791697,
					-73.33333333333326
				],
				[
					-58.888997395833485,
					-78.88891601562494
				],
				[
					-61.111165364583485,
					-81.11112467447913
				],
				[
					-61.111165364583485,
					-82.22224934895826
				],
				[
					-62.22233072916697,
					-82.22224934895826
				],
				[
					-63.333333333333485,
					-82.22224934895826
				],
				[
					-78.88899739583348,
					-82.22224934895826
				],
				[
					-111.11116536458348,
					-82.22224934895826
				],
				[
					-198.88899739583348,
					-82.22224934895826
				],
				[
					-274.44449869791697,
					-82.22224934895826
				],
				[
					-350,
					-82.22224934895826
				],
				[
					-406.66666666666697,
					-86.66666666666663
				],
				[
					-440,
					-89.99999999999994
				],
				[
					-460,
					-89.99999999999994
				],
				[
					-474.44449869791697,
					-89.99999999999994
				],
				[
					-491.1111653645835,
					-89.99999999999994
				],
				[
					-507.77783203125,
					-89.99999999999994
				],
				[
					-525.555582682292,
					-89.99999999999994
				],
				[
					-547.77783203125,
					-89.99999999999994
				],
				[
					-567.77783203125,
					-89.99999999999994
				],
				[
					-593.3333333333335,
					-89.99999999999994
				],
				[
					-601.1111653645835,
					-89.99999999999994
				],
				[
					-610,
					-89.99999999999994
				],
				[
					-625.555582682292,
					-89.99999999999994
				],
				[
					-644.444498697917,
					-89.99999999999994
				],
				[
					-667.77783203125,
					-89.99999999999994
				],
				[
					-691.1111653645835,
					-89.99999999999994
				],
				[
					-705.555582682292,
					-89.99999999999994
				],
				[
					-714.444498697917,
					-89.99999999999994
				],
				[
					-715.555582682292,
					-89.99999999999994
				],
				[
					-717.77783203125,
					-87.77779134114576
				],
				[
					-720,
					-87.77779134114576
				],
				[
					-721.1111653645835,
					-87.77779134114576
				],
				[
					-722.2222493489585,
					-87.77779134114576
				],
				[
					-728.888916015625,
					-87.77779134114576
				],
				[
					-736.6666666666665,
					-85.55558268229163
				],
				[
					-747.77783203125,
					-84.44445800781244
				],
				[
					-751.1111653645835,
					-83.33333333333326
				],
				[
					-753.3333333333335,
					-83.33333333333326
				],
				[
					-755.5555826822915,
					-83.33333333333326
				],
				[
					-768.888916015625,
					-83.33333333333326
				],
				[
					-780,
					-83.33333333333326
				],
				[
					-781.1111653645835,
					-83.33333333333326
				],
				[
					-782.2222493489585,
					-83.33333333333326
				],
				[
					-783.3333333333335,
					-83.33333333333326
				],
				[
					-790,
					-81.11112467447913
				],
				[
					-792.2222493489585,
					-81.11112467447913
				],
				[
					-793.3333333333335,
					-81.11112467447913
				],
				[
					-796.6666666666665,
					-81.11112467447913
				],
				[
					-803.3333333333335,
					-81.11112467447913
				],
				[
					-820,
					-81.11112467447913
				],
				[
					-825.5555826822915,
					-81.11112467447913
				],
				[
					-826.6666666666665,
					-81.11112467447913
				],
				[
					-827.77783203125,
					-81.11112467447913
				],
				[
					-830,
					-81.11112467447913
				],
				[
					-833.3333333333335,
					-81.11112467447913
				],
				[
					-834.4444986979165,
					-81.11112467447913
				],
				[
					-836.6666666666665,
					-81.11112467447913
				],
				[
					-837.77783203125,
					-79.99999999999994
				],
				[
					-840,
					-78.88891601562494
				],
				[
					-843.3333333333335,
					-68.88891601562494
				],
				[
					-855.5555826822915,
					-53.333333333333314
				],
				[
					-871.1111653645835,
					-31.11112467447913
				],
				[
					-874.4444986979165,
					-28.888916015625
				],
				[
					-876.6666666666665,
					-23.333333333333258
				],
				[
					-876.6666666666665,
					-22.222249348958258
				],
				[
					-876.6666666666665,
					-21.11112467447913
				],
				[
					-876.6666666666665,
					-18.888916015625
				],
				[
					-876.6666666666665,
					-17.777791341145758
				],
				[
					-876.6666666666665,
					-16.66666666666663
				],
				[
					-877.77783203125,
					-15.555582682291629
				],
				[
					-878.888916015625,
					-13.333333333333258
				],
				[
					-878.888916015625,
					-11.111124674479129
				],
				[
					-880,
					-6.666666666666629
				],
				[
					-881.1111653645835,
					-4.4444580078125
				],
				[
					-881.1111653645835,
					-4.4444580078125
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "freedraw",
			"version": 288,
			"versionNonce": 1996528581,
			"isDeleted": false,
			"id": "tHJGsZHjWHYlqBVdl_Iqe",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3186.649441273841,
			"y": -572.7332431251838,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 54.444417317708485,
			"height": 22.22220865885413,
			"seed": 961977863,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					2.222249348958485,
					0
				],
				[
					4.444417317708485,
					1.1111246744791288
				],
				[
					6.666666666666515,
					6.666666666666629
				],
				[
					8.888916015625,
					12.222208658854129
				],
				[
					11.111083984375,
					18.888875325520758
				],
				[
					13.333333333333485,
					21.11112467447913
				],
				[
					13.333333333333485,
					22.22220865885413
				],
				[
					14.444417317708485,
					22.22220865885413
				],
				[
					15.555582682291515,
					22.22220865885413
				],
				[
					18.888916015625,
					18.888875325520758
				],
				[
					26.666666666666515,
					12.222208658854129
				],
				[
					40,
					5.5555419921875
				],
				[
					52.222249348958485,
					1.1111246744791288
				],
				[
					54.444417317708485,
					1.1111246744791288
				],
				[
					54.444417317708485,
					1.1111246744791288
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "text",
			"version": 442,
			"versionNonce": 1807462155,
			"isDeleted": false,
			"id": "QOQHlxfF",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3584.9333768207157,
			"y": -685.3443271095588,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 124.501953125,
			"height": 23,
			"seed": 486753575,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "return List<T>",
			"rawText": "return List<T>",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "return List<T>",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "freedraw",
			"version": 331,
			"versionNonce": 40033061,
			"isDeleted": false,
			"id": "Oc24qYv5Kfd6mujtiBk9j",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2506.689204895635,
			"y": -51.76468092566472,
			"strokeColor": "#e03131",
			"backgroundColor": "transparent",
			"width": 1214.102595402644,
			"height": 74.35894305889417,
			"seed": 785254471,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					2.5641338641826223,
					0
				],
				[
					5.128173828125,
					0
				],
				[
					10.256441556490472,
					3.846153846153925
				],
				[
					23.076923076923094,
					10.25634765625
				],
				[
					43.58971228966357,
					15.384615384615472
				],
				[
					64.10259540264428,
					23.076923076923094
				],
				[
					84.61538461538476,
					28.205096905048094
				],
				[
					108.97432767427881,
					34.61538461538464
				],
				[
					139.74355844350953,
					43.58971228966345
				],
				[
					174.35894305889428,
					46.15384615384619
				],
				[
					211.53846153846143,
					46.15384615384619
				],
				[
					260.25644155649024,
					46.15384615384619
				],
				[
					288.46153846153857,
					42.30769230769238
				],
				[
					310.25644155649024,
					35.89740459735583
				],
				[
					335.8974045973557,
					30.76923076923083
				],
				[
					366.66663536658643,
					25.640963040865472
				],
				[
					398.7179800180288,
					24.358943058894283
				],
				[
					433.33336463341334,
					24.358943058894283
				],
				[
					469.23076923076906,
					24.358943058894283
				],
				[
					506.41028771033643,
					24.358943058894283
				],
				[
					538.4615384615381,
					25.640963040865472
				],
				[
					569.230769230769,
					30.76923076923083
				],
				[
					594.871826171875,
					39.74355844350964
				],
				[
					620.5127892127402,
					48.71788611778845
				],
				[
					625.6410569411055,
					51.28201998197119
				],
				[
					637.1795184795674,
					62.820481520432736
				],
				[
					644.871826171875,
					70.51278921274036
				],
				[
					648.7179800180288,
					74.35894305889417
				],
				[
					648.7179800180288,
					73.07692307692321
				],
				[
					648.7179800180288,
					60.256347656250114
				],
				[
					648.7179800180288,
					44.87173227163464
				],
				[
					652.5641338641826,
					34.61538461538464
				],
				[
					656.4102877103364,
					25.640963040865472
				],
				[
					665.3846153846152,
					15.384615384615472
				],
				[
					682.0512507512017,
					10.25634765625
				],
				[
					698.7179800180288,
					6.410193810096189
				],
				[
					723.0768291766826,
					5.128173828125
				],
				[
					744.871826171875,
					3.846153846153925
				],
				[
					787.1795184795674,
					3.846153846153925
				],
				[
					806.4102877103364,
					3.846153846153925
				],
				[
					838.4614445612979,
					3.846153846153925
				],
				[
					888.4614445612979,
					10.25634765625
				],
				[
					919.2306753305284,
					17.948655348557736
				],
				[
					952.5641338641826,
					25.640963040865472
				],
				[
					978.2050969050479,
					32.05125075120202
				],
				[
					1006.4102877103364,
					33.333270733173094
				],
				[
					1038.4614445612979,
					33.333270733173094
				],
				[
					1082.0512507512017,
					32.05125075120202
				],
				[
					1130.7691368689902,
					25.640963040865472
				],
				[
					1169.2306753305284,
					16.666635366586547
				],
				[
					1189.7435584435093,
					12.820481520432736
				],
				[
					1196.1537522536055,
					10.25634765625
				],
				[
					1198.7179800180284,
					8.974327674278925
				],
				[
					1205.128173828125,
					5.128173828125
				],
				[
					1207.6922137920674,
					3.846153846153925
				],
				[
					1210.2564415564902,
					2.5640399639423777
				],
				[
					1212.8204815204326,
					1.2820199819711888
				],
				[
					1214.102595402644,
					0
				],
				[
					1214.102595402644,
					0
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "text",
			"version": 334,
			"versionNonce": 1848087979,
			"isDeleted": false,
			"id": "BUhYa1FO",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3023.355840262221,
			"y": 39.26099140005647,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 256.81640625,
			"height": 23,
			"seed": 1016293223,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Convert to List<Board> here!",
			"rawText": "Convert to List<Board> here!",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Convert to List<Board> here!",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 1034,
			"versionNonce": 436612741,
			"isDeleted": false,
			"id": "4Zyks4FhkoVsfz0zc5uUl",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5406.153614328692,
			"y": -540.8956209278585,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 188.45624267711275,
			"height": 0.8288279036566948,
			"seed": 423607049,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "eR6QkmahUMZjD8dZ_FP94",
				"focus": -0.07275433795116285,
				"gap": 2.473164249160618
			},
			"endBinding": {
				"elementId": "_qGuRyf1FkOj1__b7AMe-",
				"focus": -0.008337899019808767,
				"gap": 15.762084960937045
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					188.45624267711275,
					-0.8288279036566948
				]
			]
		},
		{
			"type": "text",
			"version": 276,
			"versionNonce": 329099339,
			"isDeleted": false,
			"id": "a0vWRJrr",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5460.324264790403,
			"y": -574.8424397374506,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 58.92578125,
			"height": 23,
			"seed": 1367699945,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "4Zyks4FhkoVsfz0zc5uUl",
					"type": "arrow"
				}
			],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Invoke",
			"rawText": "Invoke",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Invoke",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "image",
			"version": 304,
			"versionNonce": 1017648613,
			"isDeleted": false,
			"id": "_qGuRyf1FkOj1__b7AMe-",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5610.371941966742,
			"y": -599.46142777456,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 693.1126552954653,
			"height": 111.33331298828126,
			"seed": 699485385,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "4Zyks4FhkoVsfz0zc5uUl",
					"type": "arrow"
				},
				{
					"id": "EgzH8ngoSaI-uxB70hB6G",
					"type": "arrow"
				}
			],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "73584240d8d994d39149a886c54857279059c084",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 974,
			"versionNonce": 1375233771,
			"isDeleted": false,
			"id": "lipeLXxyA1bY-kLwXihmV",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6838.770596313114,
			"y": -536.1389681663427,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 232.49999999999994,
			"height": 0,
			"seed": 157207465,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "QxpIHkpk",
				"focus": -0.20671477253577905,
				"gap": 12.123505691170067
			},
			"endBinding": {
				"elementId": "HXECpZR2",
				"focus": -0.01110232986335177,
				"gap": 11.036689758300781
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					232.49999999999994,
					0
				]
			]
		},
		{
			"type": "text",
			"version": 525,
			"versionNonce": 894832965,
			"isDeleted": false,
			"id": "i4awtjXx",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6915.971096801395,
			"y": -565.9444343013533,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 72.265625,
			"height": 23,
			"seed": 1488547465,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Execute",
			"rawText": "Execute",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Execute",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 481,
			"versionNonce": 2143441291,
			"isDeleted": false,
			"id": "Uw4BUYMU",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6923.770596313114,
			"y": -526.1111213130721,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 54.462890625,
			"height": 23,
			"seed": 1853666665,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Async",
			"rawText": "Async",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Async",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 575,
			"versionNonce": 1711153097,
			"isDeleted": false,
			"id": "HXECpZR2",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7082.307286071415,
			"y": -548.7777472896346,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 129.13987731933594,
			"height": 25,
			"seed": 1039409225,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "lipeLXxyA1bY-kLwXihmV",
					"type": "arrow"
				},
				{
					"id": "efebBXZjMkifMlT4poIAe",
					"type": "arrow"
				}
			],
			"updated": 1715236726359,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "Showindicator",
			"rawText": "Showindicator",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Showindicator",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 591,
			"versionNonce": 1043720235,
			"isDeleted": false,
			"id": "pyGEPgMe",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7305.841907062132,
			"y": -524.2367126978279,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 48.544921875,
			"height": 23,
			"seed": 1157312297,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Await",
			"rawText": "Await",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Await",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 904,
			"versionNonce": 1453081605,
			"isDeleted": false,
			"id": "efebBXZjMkifMlT4poIAe",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7220.8687800554,
			"y": -536.6372466098774,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 232.49999999999994,
			"height": 0,
			"seed": 972907017,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619439,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "HXECpZR2",
				"focus": -0.028759945619422068,
				"gap": 9.421616664649264
			},
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					232.49999999999994,
					0
				]
			]
		},
		{
			"type": "arrow",
			"version": 592,
			"versionNonce": 591243979,
			"isDeleted": false,
			"id": "EgzH8ngoSaI-uxB70hB6G",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6318.084358690586,
			"y": -537.1295191125764,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 214.33100919003203,
			"height": 2.3373287097545443,
			"seed": 2046179561,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "_qGuRyf1FkOj1__b7AMe-",
				"focus": 0.045869620342952375,
				"gap": 14.599761428378315
			},
			"endBinding": {
				"elementId": "QxpIHkpk",
				"focus": -0.02882966609208104,
				"gap": 7.031954674919689
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					214.33100919003203,
					2.3373287097545443
				]
			]
		},
		{
			"type": "text",
			"version": 318,
			"versionNonce": 2027912037,
			"isDeleted": false,
			"id": "AI4JHQSE",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6389.662420284627,
			"y": -570.2486013088683,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 63.37890625,
			"height": 23,
			"seed": 913388489,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Handle",
			"rawText": "Handle",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Handle",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 615,
			"versionNonce": 407592263,
			"isDeleted": false,
			"id": "QxpIHkpk",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6539.4473225555375,
			"y": -546.0550335096455,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 287.19976806640625,
			"height": 25,
			"seed": 1669976745,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "EgzH8ngoSaI-uxB70hB6G",
					"type": "arrow"
				},
				{
					"id": "lipeLXxyA1bY-kLwXihmV",
					"type": "arrow"
				}
			],
			"updated": 1715236726359,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "PCBSelectionChangedCommand",
			"rawText": "PCBSelectionChangedCommand",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "PCBSelectionChangedCommand",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "freedraw",
			"version": 283,
			"versionNonce": 277137093,
			"isDeleted": false,
			"id": "6TxUfzs0cbrRUm65KXcwZ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6212.2398360849975,
			"y": -475.3327841606874,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 504.44441731770814,
			"height": 166.66666666666663,
			"seed": 1709938057,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0,
					2.2222493489582575
				],
				[
					0,
					8.888916015625
				],
				[
					0,
					20
				],
				[
					1.111083984375,
					31.111083984375
				],
				[
					1.111083984375,
					45.55558268229163
				],
				[
					2.2222493489583712,
					57.77775065104163
				],
				[
					2.2222493489583712,
					70
				],
				[
					2.2222493489583712,
					88.888916015625
				],
				[
					2.2222493489583712,
					98.88891601562489
				],
				[
					2.2222493489583712,
					103.33333333333326
				],
				[
					2.2222493489583712,
					108.88891601562489
				],
				[
					2.2222493489583712,
					114.44441731770826
				],
				[
					2.2222493489583712,
					116.66666666666663
				],
				[
					2.2222493489583712,
					118.88891601562489
				],
				[
					2.2222493489583712,
					119.99999999999989
				],
				[
					2.2222493489583712,
					124.44441731770826
				],
				[
					2.2222493489583712,
					136.66666666666663
				],
				[
					2.2222493489583712,
					143.33333333333326
				],
				[
					3.3333333333333712,
					144.44441731770826
				],
				[
					4.444417317708371,
					144.44441731770826
				],
				[
					14.444417317708371,
					144.44441731770826
				],
				[
					43.33333333333337,
					144.44441731770826
				],
				[
					111.111083984375,
					147.77775065104163
				],
				[
					184.44441731770826,
					158.8889160156249
				],
				[
					249.9999999999999,
					163.33333333333326
				],
				[
					316.66658528645814,
					165.55558268229163
				],
				[
					396.66658528645814,
					166.66666666666663
				],
				[
					438.8889160156249,
					166.66666666666663
				],
				[
					466.66658528645814,
					166.66666666666663
				],
				[
					483.3332519531249,
					166.66666666666663
				],
				[
					487.77775065104163,
					166.66666666666663
				],
				[
					489.99991861979163,
					166.66666666666663
				],
				[
					492.22224934895814,
					166.66666666666663
				],
				[
					493.3332519531249,
					166.66666666666663
				],
				[
					495.55558268229163,
					166.66666666666663
				],
				[
					499.99991861979163,
					166.66666666666663
				],
				[
					504.44441731770814,
					166.66666666666663
				],
				[
					504.44441731770814,
					165.55558268229163
				],
				[
					504.44441731770814,
					163.33333333333326
				],
				[
					501.11108398437466,
					144.44441731770826
				],
				[
					498.88891601562466,
					122.22224934895826
				],
				[
					498.88891601562466,
					99.99999999999989
				],
				[
					497.77775065104163,
					81.111083984375
				],
				[
					496.66658528645814,
					60
				],
				[
					495.55558268229163,
					41.111083984375
				],
				[
					495.55558268229163,
					24.444417317708258
				],
				[
					495.55558268229163,
					12.222249348958258
				],
				[
					495.55558268229163,
					11.111083984375
				],
				[
					495.55558268229163,
					10
				],
				[
					495.55558268229163,
					7.777750651041629
				],
				[
					495.55558268229163,
					6.666666666666629
				],
				[
					495.55558268229163,
					5.555582682291629
				],
				[
					495.55558268229163,
					3.3333333333332575
				],
				[
					495.55558268229163,
					1.111083984375
				],
				[
					495.55558268229163,
					0
				],
				[
					495.55558268229163,
					0
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "freedraw",
			"version": 239,
			"versionNonce": 1008972811,
			"isDeleted": false,
			"id": "jYwtZ9cVrx-cOrrUxC-tW",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6681.1287521006225,
			"y": -476.4438681450624,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 45.55550130208326,
			"height": 31.11116536458337,
			"seed": 13761641,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					2.22216796875,
					-1.1111653645833712
				],
				[
					8.888834635416742,
					-7.77783203125
				],
				[
					17.777669270833258,
					-20
				],
				[
					24.4443359375,
					-31.11116536458337
				],
				[
					26.666666666666742,
					-31.11116536458337
				],
				[
					28.888834635416742,
					-30
				],
				[
					29.999999999999773,
					-28.888916015625
				],
				[
					31.111002604166742,
					-26.666666666666742
				],
				[
					37.77766927083326,
					-17.77783203125
				],
				[
					43.33333333333326,
					-8.888916015625
				],
				[
					45.55550130208326,
					-4.4444986979167425
				],
				[
					45.55550130208326,
					-4.4444986979167425
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "text",
			"version": 291,
			"versionNonce": 1344906789,
			"isDeleted": false,
			"id": "ELsUPh1H",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6333.351082829789,
			"y": -290.88836684297905,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 309.04296875,
			"height": 23,
			"seed": 67667785,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Pass SelectedItem (board) as para",
			"rawText": "Pass SelectedItem (board) as para",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Pass SelectedItem (board) as para",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 648,
			"versionNonce": 1788427945,
			"isDeleted": false,
			"id": "ay2axinY",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7479.8921467742875,
			"y": -552.2772421684999,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 201.37982177734375,
			"height": 25,
			"seed": 971683369,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "XybZjl1-XgC0AVjCckF-u",
					"type": "arrow"
				}
			],
			"updated": 1715236726360,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "SelectedPCBChanged",
			"rawText": "SelectedPCBChanged",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "SelectedPCBChanged",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 538,
			"versionNonce": 985373061,
			"isDeleted": false,
			"id": "ldkkglYisUHMhaqnGSDcC",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7625.038508371027,
			"y": -569.1982287361093,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 141.0645003695763,
			"height": 202.49539935295843,
			"seed": 358877449,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "KFsZ49sS",
				"focus": 0.7866926865074481,
				"gap": 15.893765355634969
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					141.0645003695763,
					-202.49539935295843
				]
			]
		},
		{
			"type": "arrow",
			"version": 457,
			"versionNonce": 327707979,
			"isDeleted": false,
			"id": "XybZjl1-XgC0AVjCckF-u",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7614.743140555756,
			"y": -512.8201628301949,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 128.55265546278224,
			"height": 80.28291370363593,
			"seed": 535822313,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "ay2axinY",
				"focus": 0.07459329515415113,
				"gap": 14.457079338304993
			},
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					128.55265546278224,
					80.28291370363593
				]
			]
		},
		{
			"type": "text",
			"version": 327,
			"versionNonce": 177277737,
			"isDeleted": false,
			"id": "1nERSNAi",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7816.4821789472135,
			"y": -793.9186106876759,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 222.353515625,
			"height": 23,
			"seed": 664410825,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236730699,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "If the board object is null ",
			"rawText": "If the board object is null ",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "If the board object is null ",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 312,
			"versionNonce": 1916963819,
			"isDeleted": false,
			"id": "ZadP42a1",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7792.3964116058305,
			"y": -425.94382745495807,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 255.712890625,
			"height": 23,
			"seed": 754833833,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "E7tyDkZzwv23jJHUhIiuk",
					"type": "arrow"
				}
			],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "If the board object is not null ",
			"rawText": "If the board object is not null ",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "If the board object is not null ",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "image",
			"version": 526,
			"versionNonce": 837598277,
			"isDeleted": false,
			"id": "nvi-Yl6aZ1AWUdHnxFPjV",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6856.473896087981,
			"y": -293.41260163122104,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 1170.4250694373982,
			"height": 910.3306095624208,
			"seed": 289848457,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "5251fddcb9b586300f9a22dfec78a449b182e494",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 848,
			"versionNonce": 1031788171,
			"isDeleted": false,
			"id": "E7tyDkZzwv23jJHUhIiuk",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8055.902559377545,
			"y": -415.4607830124959,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 245.39639422070422,
			"height": 111.29573683630656,
			"seed": 70271849,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "ZadP42a1",
				"focus": 0.8707325967731255,
				"gap": 7.793257146714495
			},
			"endBinding": {
				"elementId": "R5zTwzuK",
				"focus": 0.7818056023191962,
				"gap": 10.190774936869275
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					245.39639422070422,
					-111.29573683630656
				]
			]
		},
		{
			"type": "text",
			"version": 595,
			"versionNonce": 1006037925,
			"isDeleted": false,
			"id": "R5zTwzuK",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8311.489728535118,
			"y": -556.8607243771212,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 413.583984375,
			"height": 46,
			"seed": 839249481,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "E7tyDkZzwv23jJHUhIiuk",
					"type": "arrow"
				},
				{
					"id": "v3YpdABjT8PGmETvy7enn",
					"type": "arrow"
				},
				{
					"id": "z7fNWFEZ0udAA4MNtV5tP",
					"type": "arrow"
				},
				{
					"id": "NzlOvUP9t0gfITzrmbV5i",
					"type": "arrow"
				}
			],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "If the PCBItems source already got populated\nand the selected PCB is already selected once",
			"rawText": "If the PCBItems source already got populated\nand the selected PCB is already selected once",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "If the PCBItems source already got populated\nand the selected PCB is already selected once",
			"lineHeight": 1.15,
			"baseline": 41
		},
		{
			"type": "arrow",
			"version": 376,
			"versionNonce": 1408945451,
			"isDeleted": false,
			"id": "4SOrBGZTc7RTjBWdTesYY",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8057.460301182906,
			"y": -385.2135722527786,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 280.303067294034,
			"height": 115.15158913352275,
			"seed": 832781609,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					280.303067294034,
					115.15158913352275
				]
			]
		},
		{
			"type": "text",
			"version": 489,
			"versionNonce": 398733061,
			"isDeleted": false,
			"id": "94PFc95c",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8362.942157014066,
			"y": -279.81949963434124,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 285.654296875,
			"height": 69,
			"seed": 846442505,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "cCF-8RWubRIZ6W8a1k_TF",
					"type": "arrow"
				},
				{
					"id": "K3mTzXc65nkTt9CWsLhyz",
					"type": "arrow"
				},
				{
					"id": "xo-1AKMCtszoHiJFN8Tj3",
					"type": "arrow"
				}
			],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "If the PCBItems source is empty\nor the selected PCB is a newly\nselected one",
			"rawText": "If the PCBItems source is empty\nor the selected PCB is a newly\nselected one",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "If the PCBItems source is empty\nor the selected PCB is a newly\nselected one",
			"lineHeight": 1.15,
			"baseline": 64
		},
		{
			"type": "text",
			"version": 433,
			"versionNonce": 362683495,
			"isDeleted": false,
			"id": "aUAOwCMw",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9204.510708596625,
			"y": -179.8748928918675,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 190.73985290527344,
			"height": 25,
			"seed": 1785906921,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "C_EIWz5XFoc-JKMHqw_dx",
					"type": "arrow"
				},
				{
					"id": "PX9H9QVas5yswJ2jKKkN6",
					"type": "arrow"
				}
			],
			"updated": 1715236726361,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "GetBoardInfoAsync",
			"rawText": "GetBoardInfoAsync",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "GetBoardInfoAsync",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 446,
			"versionNonce": 1137837669,
			"isDeleted": false,
			"id": "PX9H9QVas5yswJ2jKKkN6",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8670.8408557715,
			"y": -267.50114472908706,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 520.4967583550351,
			"height": 101.17834226049558,
			"seed": 1143734729,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "aUAOwCMw",
				"focus": -0.7136736775002013,
				"gap": 13.173094470090291
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					520.4967583550351,
					101.17834226049558
				]
			]
		},
		{
			"type": "text",
			"version": 719,
			"versionNonce": 520339051,
			"isDeleted": false,
			"id": "RC8VcsTK",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0.18738428564501763,
			"x": 8957.40656969681,
			"y": -200.99685514045484,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 48.544921875,
			"height": 23,
			"seed": 1287315625,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Await",
			"rawText": "Await",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Await",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 540,
			"versionNonce": 808288649,
			"isDeleted": false,
			"id": "CBSuFFtq",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10184.937838371536,
			"y": -271.2044013689243,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 220.71981811523438,
			"height": 25,
			"seed": 1640686473,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "uTAROrpsW4JmnWnr0x4hF",
					"type": "arrow"
				},
				{
					"id": "nJVn8xX_v6mD7NJbmU37o",
					"type": "arrow"
				}
			],
			"updated": 1715236726361,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "CreateBoardInfoAsync",
			"rawText": "CreateBoardInfoAsync",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "CreateBoardInfoAsync",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 805,
			"versionNonce": 1349957899,
			"isDeleted": false,
			"id": "K3mTzXc65nkTt9CWsLhyz",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8665.215711092891,
			"y": -265.0933072120232,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 1482.7775065104165,
			"height": 61.42167279567411,
			"seed": 530025065,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "94PFc95c",
				"focus": -0.12154670795944238,
				"gap": 16.619257203825327
			},
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					697.0118513955049,
					-61.42167279567411
				],
				[
					1482.7775065104165,
					-3.3332722981770644
				]
			]
		},
		{
			"type": "text",
			"version": 861,
			"versionNonce": 573927717,
			"isDeleted": false,
			"id": "2DuE1arC",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9320.053995936301,
			"y": -318.18136469287253,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 48.544921875,
			"height": 23,
			"seed": 250369353,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Await",
			"rawText": "Await",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Await",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 865,
			"versionNonce": 1525768107,
			"isDeleted": false,
			"id": "C_EIWz5XFoc-JKMHqw_dx",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9255.343537462837,
			"y": -142.7248905843336,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 134.67859424172184,
			"height": 113.20349402929969,
			"seed": 321176617,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "aUAOwCMw",
				"focus": 0.13797836474052969,
				"gap": 12.150002307533896
			},
			"endBinding": {
				"elementId": "g5wl2PHy",
				"focus": -0.10356129140404158,
				"gap": 15.351525720373274
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-134.67859424172184,
					113.20349402929969
				]
			]
		},
		{
			"type": "text",
			"version": 496,
			"versionNonce": 253101189,
			"isDeleted": false,
			"id": "g5wl2PHy",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9004.996725422305,
			"y": -14.169870834660628,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 162.587890625,
			"height": 46,
			"seed": 405559049,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "C_EIWz5XFoc-JKMHqw_dx",
					"type": "arrow"
				},
				{
					"id": "uTAROrpsW4JmnWnr0x4hF",
					"type": "arrow"
				}
			],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "CONDITIONDATA\n        (byte)",
			"rawText": "CONDITIONDATA\n        (byte)",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "CONDITIONDATA\n        (byte)",
			"lineHeight": 1.15,
			"baseline": 41
		},
		{
			"type": "text",
			"version": 552,
			"versionNonce": 413020747,
			"isDeleted": false,
			"id": "APLGAxt9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9040.615114059827,
			"y": -145.94289952599718,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 172.32421875,
			"height": 69,
			"seed": 8293865,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Get by passing the \nselected PCB's \nConditionGUID",
			"rawText": "Get by passing the \nselected PCB's \nConditionGUID",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Get by passing the \nselected PCB's \nConditionGUID",
			"lineHeight": 1.15,
			"baseline": 64
		},
		{
			"type": "arrow",
			"version": 976,
			"versionNonce": 1833774053,
			"isDeleted": false,
			"id": "uTAROrpsW4JmnWnr0x4hF",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9175.22563050973,
			"y": 1.5122031143324648,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 998.6458935758251,
			"height": 257.3783047545023,
			"seed": 1395058889,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "g5wl2PHy",
				"focus": 0.35500450943424755,
				"gap": 7.641014462426028
			},
			"endBinding": {
				"elementId": "CBSuFFtq",
				"focus": 0.695032571586311,
				"gap": 11.066314285979388
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					998.6458935758251,
					-257.3783047545023
				]
			]
		},
		{
			"type": "text",
			"version": 523,
			"versionNonce": 978825451,
			"isDeleted": false,
			"id": "lMd6Wv6S",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9551.029097306606,
			"y": -75.08339215627507,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 340.1953125,
			"height": 23,
			"seed": 700508073,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Parse (convert to readable data) using",
			"rawText": "Parse (convert to readable data) using",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Parse (convert to readable data) using",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 841,
			"versionNonce": 2095605573,
			"isDeleted": false,
			"id": "fHvlBYFA",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0.20150763699617524,
			"x": 8891.5553941625,
			"y": -236.76457218884286,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 240.146484375,
			"height": 23,
			"seed": 1584925321,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Pass selected board object",
			"rawText": "Pass selected board object",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Pass selected board object",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "image",
			"version": 502,
			"versionNonce": 1906858891,
			"isDeleted": false,
			"id": "PNqljnXN-lQ1N8lIkQUuk",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5260.8253175327745,
			"y": 179.2282140013249,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 1240,
			"height": 372,
			"seed": 2047332713,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "b55993223943985fab03f8d737dd5d468c0a8a0e",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 487,
			"versionNonce": 541406885,
			"isDeleted": false,
			"id": "tRVDMI05mzPIE0rvlIZXX",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5330.279674951414,
			"y": 601.0660732407111,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 1115,
			"height": 113,
			"seed": 1639693385,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "36abd36d2c8c2556ba4b2f0896ef6671032bde84",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 723,
			"versionNonce": 1523233323,
			"isDeleted": false,
			"id": "nJVn8xX_v6mD7NJbmU37o",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10293.234489941504,
			"y": -239.80439130499383,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 3.028587189529844,
			"height": 213.20342621245933,
			"seed": 1164921641,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "CBSuFFtq",
				"focus": 0.021094520621579422,
				"gap": 6.400010063930495
			},
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					3.028587189529844,
					213.20342621245933
				]
			]
		},
		{
			"type": "text",
			"version": 856,
			"versionNonce": 303110661,
			"isDeleted": false,
			"id": "dfgc8ZM4",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10140.549218458313,
			"y": -162.84908695858792,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 497.24609375,
			"height": 23,
			"seed": 1301151241,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Create BoardInfo = Selected Board + CONDITIONDATA",
			"rawText": "Create BoardInfo = Selected Board + CONDITIONDATA",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Create BoardInfo = Selected Board + CONDITIONDATA",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 580,
			"versionNonce": 648623307,
			"isDeleted": false,
			"id": "M6Ep6cSp",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10253.635470155485,
			"y": -5.254624816555975,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 86.728515625,
			"height": 23,
			"seed": 152261865,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "TUmHArQrHQSPp0NVTD0IV",
					"type": "arrow"
				}
			],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "BoardInfo",
			"rawText": "BoardInfo",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "BoardInfo",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 499,
			"versionNonce": 1531548005,
			"isDeleted": false,
			"id": "xo-1AKMCtszoHiJFN8Tj3",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8664.93295814926,
			"y": -266.27209320157783,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 2186.796653053977,
			"height": 201.606552234116,
			"seed": 462081993,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "94PFc95c",
				"focus": 0.13343552843817427,
				"gap": 16.336504260194488
			},
			"endBinding": {
				"elementId": "dw1tHtsj",
				"focus": -0.5173791459249107,
				"gap": 12.727272727272066
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					1053.048475711903,
					-192.0827136459464
				],
				[
					2186.796653053977,
					9.52383858816961
				]
			]
		},
		{
			"type": "text",
			"version": 266,
			"versionNonce": 1418459015,
			"isDeleted": false,
			"id": "dw1tHtsj",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10864.45688393051,
			"y": -265.8391082178115,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 177.41983032226562,
			"height": 25,
			"seed": 722464425,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "xo-1AKMCtszoHiJFN8Tj3",
					"type": "arrow"
				},
				{
					"id": "l-b5AwqOwKs1r3Xks4yy1",
					"type": "arrow"
				},
				{
					"id": "TUmHArQrHQSPp0NVTD0IV",
					"type": "arrow"
				},
				{
					"id": "v3YpdABjT8PGmETvy7enn",
					"type": "arrow"
				},
				{
					"id": "2IfpCidnZMC4UbUDvj1Ma",
					"type": "arrow"
				}
			],
			"updated": 1715236726363,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "inspResultUpdater",
			"rawText": "inspResultUpdater",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "inspResultUpdater",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 663,
			"versionNonce": 154934469,
			"isDeleted": false,
			"id": "FIwzE9F9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9714.158418317347,
			"y": -444.29287184106977,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 58.92578125,
			"height": 23,
			"seed": 240005513,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Invoke",
			"rawText": "Invoke",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Invoke",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 430,
			"versionNonce": 66480233,
			"isDeleted": false,
			"id": "hFi6H9bo",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 11180.880287645816,
			"y": -334.9943603204882,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 144.19989013671875,
			"height": 25,
			"seed": 981538921,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726364,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "Main ViewModel",
			"rawText": "Main ViewModel",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Main ViewModel",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "rectangle",
			"version": 662,
			"versionNonce": 595999781,
			"isDeleted": false,
			"id": "brokTuM6lzAsTogHFPfSO",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 11167.546913622378,
			"y": -347.4943603204882,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 293.1505228970864,
			"height": 151.81913261859685,
			"seed": 1903436617,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false
		},
		{
			"type": "image",
			"version": 296,
			"versionNonce": 1814480043,
			"isDeleted": false,
			"id": "XhchuxHF15CQsEsyVXcbN",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10976.281107801373,
			"y": 8.782940341720746,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 635,
			"height": 353,
			"seed": 797102633,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "da7f1d7614cd1ee17cb8c6e4c70bf1be69a7ad21",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 589,
			"versionNonce": 1645571973,
			"isDeleted": false,
			"id": "l-b5AwqOwKs1r3Xks4yy1",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 11051.742611292058,
			"y": -249.71045918772313,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 161.6785910238441,
			"height": 3.0580274291844773,
			"seed": 1503175945,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "dw1tHtsj",
				"focus": 0.3874440509290937,
				"gap": 9.865897039282572
			},
			"endBinding": {
				"elementId": "BjQoVjeh",
				"focus": 0.18959492123419866,
				"gap": 7.647611551446971
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					161.6785910238441,
					-3.0580274291844773
				]
			]
		},
		{
			"type": "text",
			"version": 603,
			"versionNonce": 726757031,
			"isDeleted": false,
			"id": "BjQoVjeh",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 11221.068813867349,
			"y": -264.5709588416365,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 199.33984375,
			"height": 25,
			"seed": 724638697,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "l-b5AwqOwKs1r3Xks4yy1",
					"type": "arrow"
				},
				{
					"id": "-TeRFI4_a2wFe-RGlB5fg",
					"type": "arrow"
				},
				{
					"id": "wO2SynyNqEkPmZH7WsyuO",
					"type": "arrow"
				}
			],
			"updated": 1715236726364,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "UpdatedResultData",
			"rawText": "UpdatedResultData",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "UpdatedResultData",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 694,
			"versionNonce": 589772517,
			"isDeleted": false,
			"id": "nbjSAKtd",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 11090.907362669888,
			"y": -290.8102053925153,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 63.37890625,
			"height": 23,
			"seed": 1683470025,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Handle",
			"rawText": "Handle",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Handle",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 479,
			"versionNonce": 1865930219,
			"isDeleted": false,
			"id": "TUmHArQrHQSPp0NVTD0IV",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10349.577830402268,
			"y": 7.453927644438295,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 569.0475027901794,
			"height": 241.66660853794644,
			"seed": 933754281,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "M6Ep6cSp",
				"focus": 0.786789267876509,
				"gap": 9.21384462178321
			},
			"endBinding": {
				"elementId": "dw1tHtsj",
				"focus": -0.08883396476657011,
				"gap": 6.626427324303336
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					569.0475027901794,
					-241.66660853794644
				]
			]
		},
		{
			"type": "text",
			"version": 623,
			"versionNonce": 558436933,
			"isDeleted": false,
			"id": "EV7yb54Z",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10602.16558151834,
			"y": -75.95084879250385,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 66.69921875,
			"height": 23,
			"seed": 2018334857,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619440,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Pass to",
			"rawText": "Pass to",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Pass to",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 655,
			"versionNonce": 1094443147,
			"isDeleted": false,
			"id": "-TeRFI4_a2wFe-RGlB5fg",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 11426.242704662433,
			"y": -252.75201459018854,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 363.85767883782864,
			"height": 210.65441102329424,
			"seed": 134438761,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "BjQoVjeh",
				"focus": 0.8603569332543601,
				"gap": 5.834047045083935
			},
			"endBinding": {
				"elementId": "EO2MMyPS",
				"focus": 0.7940599753865352,
				"gap": 5.763886343504055
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					363.85767883782864,
					-210.65441102329424
				]
			]
		},
		{
			"type": "text",
			"version": 548,
			"versionNonce": 2143253321,
			"isDeleted": false,
			"id": "i8zg4c3f",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 11753.12543417647,
			"y": -549.6461209104112,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 174.83984375,
			"height": 25,
			"seed": 85297737,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726364,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "Current ViewModel",
			"rawText": "Current ViewModel",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Current ViewModel",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "rectangle",
			"version": 866,
			"versionNonce": 1470469931,
			"isDeleted": false,
			"id": "IYiOmGMy4qJIAngMtVTqF",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 11741.073986234765,
			"y": -565.992274756565,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 214.62475667112443,
			"height": 151.81913261859685,
			"seed": 945772841,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 717,
			"versionNonce": 1956315591,
			"isDeleted": false,
			"id": "EO2MMyPS",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 11795.864269843765,
			"y": -477.06988793444623,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 130.03990173339844,
			"height": 25,
			"seed": 1129073673,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "-TeRFI4_a2wFe-RGlB5fg",
					"type": "arrow"
				},
				{
					"id": "oOBk5_8hdpH-RtLVNHoUj",
					"type": "arrow"
				}
			],
			"updated": 1715236726365,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "UpdateBoard",
			"rawText": "UpdateBoard",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "UpdateBoard",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 294,
			"versionNonce": 1893693899,
			"isDeleted": false,
			"id": "fjaG0n5a",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 5.7652966631677725,
			"x": 11502.871112314075,
			"y": -399.95239553830334,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 196.787109375,
			"height": 23,
			"seed": 232252137,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Get BoardInfo as para",
			"rawText": "Get BoardInfo as para",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Get BoardInfo as para",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 911,
			"versionNonce": 709237861,
			"isDeleted": false,
			"id": "oOBk5_8hdpH-RtLVNHoUj",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 11933.264610828244,
			"y": -460.8880374930569,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 189.23039811906347,
			"height": 1.4910667766435495,
			"seed": 1955438025,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "EO2MMyPS",
				"focus": 0.23912077317868896,
				"gap": 7.3604392510806065
			},
			"endBinding": {
				"elementId": "A6jvNuj9GhSFezrOViT03",
				"focus": -0.037924621253021575,
				"gap": 17.71808401035898
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					189.23039811906347,
					1.4910667766435495
				]
			]
		},
		{
			"type": "text",
			"version": 355,
			"versionNonce": 436471915,
			"isDeleted": false,
			"id": "xK8JcNho",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 12152.708784005326,
			"y": -470.3339559875013,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 244.599609375,
			"height": 23,
			"seed": 914947241,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Send data to other Controls",
			"rawText": "Send data to other Controls",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Send data to other Controls",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "ellipse",
			"version": 410,
			"versionNonce": 2121222085,
			"isDeleted": false,
			"id": "A6jvNuj9GhSFezrOViT03",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 12140.208987455846,
			"y": -505.05619516393347,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 267.1297878689229,
			"height": 90.27781168619794,
			"seed": 594902921,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"id": "oOBk5_8hdpH-RtLVNHoUj",
					"type": "arrow"
				}
			],
			"updated": 1712906619441,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 551,
			"versionNonce": 2011439657,
			"isDeleted": false,
			"id": "Kz9bbQno",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9322.545404539627,
			"y": 192.3064283726219,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 275.43975830078125,
			"height": 25,
			"seed": 160343657,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "cCF-8RWubRIZ6W8a1k_TF",
					"type": "arrow"
				},
				{
					"id": "QOJ7rpaXYCaAApP8Qr9wG",
					"type": "arrow"
				}
			],
			"updated": 1715236726365,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "CreateInspectionItemsAsync",
			"rawText": "CreateInspectionItemsAsync",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "CreateInspectionItemsAsync",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 823,
			"versionNonce": 2132543269,
			"isDeleted": false,
			"id": "cCF-8RWubRIZ6W8a1k_TF",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8663.970462288791,
			"y": -260.3022423129977,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 648.8119402834773,
			"height": 469.1349015099678,
			"seed": 1671426377,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "94PFc95c",
				"focus": -0.9518387822307911,
				"gap": 15.374008399725426
			},
			"endBinding": {
				"elementId": "Kz9bbQno",
				"focus": -0.9350062863058496,
				"gap": 9.763001967357923
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					517.8370037884683,
					415.5147039676682
				],
				[
					648.8119402834773,
					469.1349015099678
				]
			]
		},
		{
			"type": "text",
			"version": 940,
			"versionNonce": 367471019,
			"isDeleted": false,
			"id": "689N5PR5",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9158.712110327126,
			"y": 198.54927849811304,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 48.544921875,
			"height": 23,
			"seed": 1191501865,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Await",
			"rawText": "Await",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Await",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 565,
			"versionNonce": 2126109317,
			"isDeleted": false,
			"id": "QOJ7rpaXYCaAApP8Qr9wG",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9606.49224411092,
			"y": 208.69076219878968,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 127.77777777777828,
			"height": 0,
			"seed": 296284937,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Kz9bbQno",
				"focus": 0.3107467060934232,
				"gap": 8.507081270510753
			},
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					127.77777777777828,
					0
				]
			]
		},
		{
			"type": "text",
			"version": 1073,
			"versionNonce": 1179302987,
			"isDeleted": false,
			"id": "Ht1pDiLG",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9635.92366772203,
			"y": 167.19076219878968,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 52.24609375,
			"height": 23,
			"seed": 1476897257,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "return",
			"rawText": "return",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "return",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 1078,
			"versionNonce": 637944293,
			"isDeleted": false,
			"id": "l0RfK9i3",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9751.849367591823,
			"y": 198.67228889149794,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 84.47265625,
			"height": 23,
			"seed": 321684681,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "MoV8tye755951PIbYyUtm",
					"type": "arrow"
				},
				{
					"id": "OvTr4_GoyrH4U4IVLRGOF",
					"type": "arrow"
				}
			],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "inspitems",
			"rawText": "inspitems",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "inspitems",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 870,
			"versionNonce": 1440938731,
			"isDeleted": false,
			"id": "MoV8tye755951PIbYyUtm",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9848.614880269659,
			"y": 210.84811888019442,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 131.74610925099296,
			"height": 0,
			"seed": 246612905,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "l0RfK9i3",
				"focus": 0.05876782510404155,
				"gap": 12.292856427835432
			},
			"endBinding": {
				"elementId": "UJ5o8O9Hok1PLSrAyEJXG",
				"focus": -0.35967074652251124,
				"gap": 7.236681471955308
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					131.74610925099296,
					0
				]
			]
		},
		{
			"type": "text",
			"version": 1133,
			"versionNonce": 705609029,
			"isDeleted": false,
			"id": "iykRfxGE",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9878.046303880772,
			"y": 169.34811888019442,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 64.482421875,
			"height": 23,
			"seed": 382117513,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "pass to",
			"rawText": "pass to",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "pass to",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 1183,
			"versionNonce": 1362781579,
			"isDeleted": false,
			"id": "JATfgcgq",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9995.559324714104,
			"y": 198.44869319195016,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 85.5859375,
			"height": 23,
			"seed": 1713565033,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Inspitems",
			"rawText": "Inspitems",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Inspitems",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 700,
			"versionNonce": 25506023,
			"isDeleted": false,
			"id": "dYAvdLCe",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9995.680787461097,
			"y": 123.98225603438448,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 202.4998321533203,
			"height": 25,
			"seed": 1410553929,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726367,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "DbConnect ViewModel",
			"rawText": "DbConnect ViewModel",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "DbConnect ViewModel",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "rectangle",
			"version": 1022,
			"versionNonce": 1334541355,
			"isDeleted": false,
			"id": "UJ5o8O9Hok1PLSrAyEJXG",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9987.597670992607,
			"y": 107.63610218823055,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 214.62475667112443,
			"height": 151.81913261859685,
			"seed": 1114859305,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "MoV8tye755951PIbYyUtm",
					"type": "arrow"
				}
			],
			"updated": 1712906619441,
			"link": null,
			"locked": false
		},
		{
			"type": "arrow",
			"version": 498,
			"versionNonce": 1578758149,
			"isDeleted": false,
			"id": "OvTr4_GoyrH4U4IVLRGOF",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9792.76398532413,
			"y": 229.77154571578023,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 533.7129292637146,
			"height": 312.13471129391814,
			"seed": 1269733897,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "l0RfK9i3",
				"focus": 0.5627482725126552,
				"gap": 8.099256824282293
			},
			"endBinding": {
				"elementId": "SyaoMunX",
				"focus": -0.533791643041412,
				"gap": 12.692078711885188
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					533.7129292637146,
					312.13471129391814
				]
			]
		},
		{
			"type": "arrow",
			"version": 381,
			"versionNonce": 482393803,
			"isDeleted": false,
			"id": "lNDkasenqOtUB78Cp7Wv6",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10314.185106580982,
			"y": 34.77166778609251,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 43.989341019958374,
			"height": 503.3333740234376,
			"seed": 817918185,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "SyaoMunX",
				"focus": -0.5925687953687443,
				"gap": 7.833251953125114
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					43.989341019958374,
					503.3333740234376
				]
			]
		},
		{
			"type": "text",
			"version": 667,
			"versionNonce": 466069349,
			"isDeleted": false,
			"id": "SyaoMunX",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10339.16899329973,
			"y": 545.9382937626552,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 104.51171875,
			"height": 23,
			"seed": 120875977,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "OvTr4_GoyrH4U4IVLRGOF",
					"type": "arrow"
				},
				{
					"id": "lNDkasenqOtUB78Cp7Wv6",
					"type": "arrow"
				}
			],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "boardsInfos",
			"rawText": "boardsInfos",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "boardsInfos",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 660,
			"versionNonce": 594849131,
			"isDeleted": false,
			"id": "MfnyA1Cp",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10359.83598548723,
			"y": 240.60504180953023,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 57.822265625,
			"height": 23,
			"seed": 656804521,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Add to",
			"rawText": "Add to",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Add to",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 691,
			"versionNonce": 2106028741,
			"isDeleted": false,
			"id": "RMVeK9Cq",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10008.27433997942,
			"y": 416.93829376265523,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 57.822265625,
			"height": 23,
			"seed": 1497419145,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Add to",
			"rawText": "Add to",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Add to",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 939,
			"versionNonce": 384944139,
			"isDeleted": false,
			"id": "QN6MnzzG",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10171.929491346607,
			"y": 602.9384158329677,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 525.849609375,
			"height": 69,
			"seed": 1153447017,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "[boardsInfos is a property in DbConnect ViewModel\nThis property save all data that has been created so if user \nselected it again, the program don't have to create it again]",
			"rawText": "[boardsInfos is a property in DbConnect ViewModel\nThis property save all data that has been created so if user \nselected it again, the program don't have to create it again]",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "[boardsInfos is a property in DbConnect ViewModel\nThis property save all data that has been created so if user \nselected it again, the program don't have to create it again]",
			"lineHeight": 1.15,
			"baseline": 64
		},
		{
			"type": "text",
			"version": 295,
			"versionNonce": 693105929,
			"isDeleted": false,
			"id": "pWxYmWhm",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7770.511136084653,
			"y": -438.9143525297185,
			"strokeColor": "#f08c00",
			"backgroundColor": "transparent",
			"width": 9.755996704101562,
			"height": 45,
			"seed": 493862729,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "XybZjl1-XgC0AVjCckF-u",
					"type": "arrow"
				}
			],
			"updated": 1715236726368,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "1",
			"rawText": "1",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "1",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "text",
			"version": 396,
			"versionNonce": 1268546567,
			"isDeleted": false,
			"id": "KFsZ49sS",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7781.996774096238,
			"y": -803.2324788613946,
			"strokeColor": "#f08c00",
			"backgroundColor": "transparent",
			"width": 25.631988525390625,
			"height": 45,
			"seed": 296414761,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "ldkkglYisUHMhaqnGSDcC",
					"type": "arrow"
				}
			],
			"updated": 1715236726368,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "2",
			"rawText": "2",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "2",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "arrow",
			"version": 518,
			"versionNonce": 805520773,
			"isDeleted": false,
			"id": "v3YpdABjT8PGmETvy7enn",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8730.928426974506,
			"y": -535.8335683389332,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 2208.333333333333,
			"height": 258.3333333333335,
			"seed": 2019377417,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "R5zTwzuK",
				"focus": -0.06128511889873161,
				"gap": 5.854714064387736
			},
			"endBinding": {
				"elementId": "dw1tHtsj",
				"focus": 0.7088270466780615,
				"gap": 14.438836748725748
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					1111.1110432942705,
					-2.7777099609375
				],
				[
					2208.333333333333,
					255.55562337239598
				]
			]
		},
		{
			"type": "text",
			"version": 689,
			"versionNonce": 954302795,
			"isDeleted": false,
			"id": "gNLc0dIP",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9805.354289604713,
			"y": -529.5556549274752,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 58.92578125,
			"height": 23,
			"seed": 1057109993,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Invoke",
			"rawText": "Invoke",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Invoke",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 265,
			"versionNonce": 1625967845,
			"isDeleted": false,
			"id": "KSzSVCIX",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9035.928342203455,
			"y": -654.0278381550133,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 84.51171875,
			"height": 23,
			"seed": 396710601,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "z7fNWFEZ0udAA4MNtV5tP",
					"type": "arrow"
				},
				{
					"id": "F0xf_fuWyk1JfNX-cY5av",
					"type": "arrow"
				}
			],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "boardInfo",
			"rawText": "boardInfo",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "boardInfo",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 521,
			"versionNonce": 861478891,
			"isDeleted": false,
			"id": "z7fNWFEZ0udAA4MNtV5tP",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8735.92825065072,
			"y": -542.7327740057126,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 290.0000915527344,
			"height": 97.69011666515782,
			"seed": 1726091689,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "R5zTwzuK",
				"focus": 0.695495029837725,
				"gap": 10.85453774060261
			},
			"endBinding": {
				"elementId": "KSzSVCIX",
				"focus": 0.6022320175987816,
				"gap": 10
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					290.0000915527344,
					-97.69011666515782
				]
			]
		},
		{
			"type": "text",
			"version": 328,
			"versionNonce": 574187589,
			"isDeleted": false,
			"id": "YQ2KhAqR",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 5.931675133995004,
			"x": 8780.126997552012,
			"y": -625.1496009442262,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 177.861328125,
			"height": 23,
			"seed": 1910764681,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Get from boardInfos",
			"rawText": "Get from boardInfos",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Get from boardInfos",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 519,
			"versionNonce": 1185147531,
			"isDeleted": false,
			"id": "F0xf_fuWyk1JfNX-cY5av",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9134.233470512918,
			"y": -642.8220009538941,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 1843.749847412109,
			"height": 358.9924763151647,
			"seed": 1281784681,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "KSzSVCIX",
				"focus": -0.2117377604088254,
				"gap": 13.793409559462816
			},
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					931.2498474121089,
					42.325911373758615
				],
				[
					1843.749847412109,
					358.9924763151647
				]
			]
		},
		{
			"type": "text",
			"version": 293,
			"versionNonce": 1439569829,
			"isDeleted": false,
			"id": "WqMHxdPF",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0.14512318086719578,
			"x": 10022.42769455263,
			"y": -646.3293466195237,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 66.69921875,
			"height": 23,
			"seed": 112535113,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Pass to",
			"rawText": "Pass to",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Pass to",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 340,
			"versionNonce": 1199333675,
			"isDeleted": false,
			"id": "MfpMs5tK",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9017.872093233595,
			"y": -791.0098007688612,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 85.5859375,
			"height": 23,
			"seed": 464221481,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "NzlOvUP9t0gfITzrmbV5i",
					"type": "arrow"
				},
				{
					"id": "cTElpVvWR06MXnfLqY9cn",
					"type": "arrow"
				}
			],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "inspItems",
			"rawText": "inspItems",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "inspItems",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 669,
			"versionNonce": 1171643141,
			"isDeleted": false,
			"id": "NzlOvUP9t0gfITzrmbV5i",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8726.811898702466,
			"y": -568.8470821647334,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 277.44052851817105,
			"height": 211.1041448679347,
			"seed": 515172361,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "R5zTwzuK",
				"focus": 0.8242709601872296,
				"gap": 11.986357787612178
			},
			"endBinding": {
				"elementId": "MfpMs5tK",
				"focus": 0.8249844503066728,
				"gap": 13.619666012957168
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					93.31605390612867,
					-132.2011801425891
				],
				[
					277.44052851817105,
					-211.1041448679347
				]
			]
		},
		{
			"type": "text",
			"version": 395,
			"versionNonce": 256705483,
			"isDeleted": false,
			"id": "5uLFWYVM",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 5.931675133995004,
			"x": 8763.318337911449,
			"y": -755.1100159992802,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 177.861328125,
			"height": 23,
			"seed": 701300457,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Get from boardInfos",
			"rawText": "Get from boardInfos",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Get from boardInfos",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 465,
			"versionNonce": 449720933,
			"isDeleted": false,
			"id": "cTElpVvWR06MXnfLqY9cn",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9112.435551016046,
			"y": -779.253406162491,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 228.20509690504787,
			"height": 2.5640869140625,
			"seed": 192887241,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "MfpMs5tK",
				"focus": 0.06995199271128771,
				"gap": 8.977520282451223
			},
			"endBinding": {
				"elementId": "JXgr1Z3e",
				"focus": 0.05735208496364238,
				"gap": 14.359036959136574
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					228.20509690504787,
					-2.5640869140625
				]
			]
		},
		{
			"type": "text",
			"version": 232,
			"versionNonce": 662596585,
			"isDeleted": false,
			"id": "JXgr1Z3e",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9354.99968488023,
			"y": -794.8943692033563,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 213.8397979736328,
			"height": 25,
			"seed": 887560361,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "cTElpVvWR06MXnfLqY9cn",
					"type": "arrow"
				}
			],
			"updated": 1715236726369,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "RaisePropertyChanged",
			"rawText": "RaisePropertyChanged",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "RaisePropertyChanged",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 354,
			"versionNonce": 1725784871,
			"isDeleted": false,
			"id": "Km8ZPwgs",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8500.241039248482,
			"y": -618.3784286687387,
			"strokeColor": "#f08c00",
			"backgroundColor": "transparent",
			"width": 29.375991821289062,
			"height": 45,
			"seed": 1739594633,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726369,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "1.1",
			"rawText": "1.1",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "1.1",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "text",
			"version": 384,
			"versionNonce": 175834825,
			"isDeleted": false,
			"id": "iZfu1YB9",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8482.431163760202,
			"y": -336.7118026921761,
			"strokeColor": "#f08c00",
			"backgroundColor": "transparent",
			"width": 45.251983642578125,
			"height": 45,
			"seed": 605405801,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726370,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "1.2",
			"rawText": "1.2",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "1.2",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "freedraw",
			"version": 270,
			"versionNonce": 1897307429,
			"isDeleted": false,
			"id": "-zi1_6VMcsdHUlgsL5Cib",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6884.897073082304,
			"y": -248.54501395519708,
			"strokeColor": "#f08c00",
			"backgroundColor": "transparent",
			"width": 22.222249348958258,
			"height": 92.22224934895837,
			"seed": 1457753417,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0,
					1.111083984375
				],
				[
					0,
					5.5555826822917425
				],
				[
					0,
					11.111083984375
				],
				[
					0,
					15.555582682291742
				],
				[
					-1.111165364583485,
					23.33333333333337
				],
				[
					-2.222249348958485,
					30
				],
				[
					-2.222249348958485,
					33.33333333333337
				],
				[
					-4.4444986979167425,
					35.55558268229174
				],
				[
					-5.5555826822917425,
					36.66666666666674
				],
				[
					-6.6666666666667425,
					36.66666666666674
				],
				[
					-8.888916015625,
					36.66666666666674
				],
				[
					-15.555582682291742,
					34.44441731770837
				],
				[
					-16.666666666666742,
					32.22224934895837
				],
				[
					-16.666666666666742,
					31.111083984375
				],
				[
					-17.77783203125,
					28.888916015625
				],
				[
					-17.77783203125,
					26.666666666666742
				],
				[
					-17.77783203125,
					24.44441731770837
				],
				[
					-17.77783203125,
					23.33333333333337
				],
				[
					-17.77783203125,
					21.111083984375
				],
				[
					-17.77783203125,
					18.888916015625
				],
				[
					-15.555582682291742,
					18.888916015625
				],
				[
					-13.333333333333485,
					18.888916015625
				],
				[
					-11.111165364583485,
					20
				],
				[
					-8.888916015625,
					22.22224934895837
				],
				[
					-6.6666666666667425,
					24.44441731770837
				],
				[
					-4.4444986979167425,
					27.777750651041742
				],
				[
					-2.222249348958485,
					31.111083984375
				],
				[
					0,
					33.33333333333337
				],
				[
					0,
					34.44441731770837
				],
				[
					0,
					35.55558268229174
				],
				[
					0,
					37.77775065104174
				],
				[
					0,
					40
				],
				[
					0,
					42.22224934895837
				],
				[
					0,
					45.55558268229174
				],
				[
					0,
					47.77775065104174
				],
				[
					0,
					50
				],
				[
					1.111083984375,
					53.33333333333337
				],
				[
					1.111083984375,
					54.44441731770837
				],
				[
					1.111083984375,
					57.77775065104174
				],
				[
					1.111083984375,
					60
				],
				[
					0,
					62.22224934895837
				],
				[
					0,
					65.55558268229174
				],
				[
					-2.222249348958485,
					70
				],
				[
					-2.222249348958485,
					74.44441731770837
				],
				[
					-3.333333333333485,
					77.77775065104174
				],
				[
					-4.4444986979167425,
					80
				],
				[
					-4.4444986979167425,
					82.22224934895837
				],
				[
					-3.333333333333485,
					82.22224934895837
				],
				[
					-2.222249348958485,
					84.44441731770837
				],
				[
					-2.222249348958485,
					85.55558268229174
				],
				[
					-2.222249348958485,
					87.77775065104174
				],
				[
					-2.222249348958485,
					88.888916015625
				],
				[
					0,
					91.111083984375
				],
				[
					1.111083984375,
					92.22224934895837
				],
				[
					2.22216796875,
					92.22224934895837
				],
				[
					3.3333333333332575,
					92.22224934895837
				],
				[
					4.4444173177082575,
					92.22224934895837
				],
				[
					4.4444173177082575,
					92.22224934895837
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "freedraw",
			"version": 263,
			"versionNonce": 1191769003,
			"isDeleted": false,
			"id": "cOdwuGYygYFGjd8A0JLzA",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6916.008157066679,
			"y": -42.98943127290545,
			"strokeColor": "#f08c00",
			"backgroundColor": "transparent",
			"width": 32.22216796875,
			"height": 138.888916015625,
			"seed": 1198849065,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0,
					-1.1111653645833712
				],
				[
					1.111083984375,
					-1.1111653645833712
				],
				[
					1.111083984375,
					1.111083984375
				],
				[
					3.3333333333332575,
					2.22216796875
				],
				[
					3.3333333333332575,
					8.888834635416629
				],
				[
					3.3333333333332575,
					24.44441731770837
				],
				[
					1.111083984375,
					37.77775065104163
				],
				[
					-3.333333333333485,
					51.111083984375
				],
				[
					-8.888916015625,
					57.77775065104163
				],
				[
					-8.888916015625,
					61.111083984375
				],
				[
					-11.111083984375,
					63.33333333333337
				],
				[
					-14.444417317708485,
					63.33333333333337
				],
				[
					-16.666666666666742,
					64.44441731770837
				],
				[
					-21.111083984375,
					64.44441731770837
				],
				[
					-23.333333333333485,
					64.44441731770837
				],
				[
					-25.555582682291742,
					63.33333333333337
				],
				[
					-27.777750651041742,
					62.22216796875
				],
				[
					-27.777750651041742,
					60
				],
				[
					-27.777750651041742,
					54.44441731770837
				],
				[
					-27.777750651041742,
					52.22216796875
				],
				[
					-26.666666666666742,
					48.88883463541663
				],
				[
					-24.444417317708485,
					46.66666666666663
				],
				[
					-20,
					45.55550130208337
				],
				[
					-15.555582682291742,
					43.33333333333337
				],
				[
					-12.222249348958485,
					43.33333333333337
				],
				[
					-7.7777506510417425,
					44.44441731770837
				],
				[
					-5.5555826822917425,
					47.77775065104163
				],
				[
					-3.333333333333485,
					53.33333333333337
				],
				[
					-1.111083984375,
					56.66666666666663
				],
				[
					0,
					62.22216796875
				],
				[
					0,
					64.44441731770837
				],
				[
					0,
					70
				],
				[
					0,
					76.66666666666663
				],
				[
					0,
					82.22216796875
				],
				[
					0,
					88.88883463541663
				],
				[
					0,
					94.44441731770837
				],
				[
					0,
					101.111083984375
				],
				[
					-2.222249348958485,
					106.66666666666663
				],
				[
					-2.222249348958485,
					110
				],
				[
					-2.222249348958485,
					114.44441731770837
				],
				[
					0,
					118.88883463541663
				],
				[
					0,
					121.111083984375
				],
				[
					0,
					123.33333333333326
				],
				[
					0,
					125.55550130208326
				],
				[
					0,
					128.88883463541663
				],
				[
					0,
					131.111083984375
				],
				[
					0,
					133.33333333333326
				],
				[
					2.2222493489582575,
					136.66666666666663
				],
				[
					3.3333333333332575,
					137.77775065104163
				],
				[
					4.4444173177082575,
					137.77775065104163
				],
				[
					4.4444173177082575,
					137.77775065104163
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "freedraw",
			"version": 289,
			"versionNonce": 1089581189,
			"isDeleted": false,
			"id": "OQGXJf0P8rExFe_e-7zal",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6922.674823733346,
			"y": 128.12165271146955,
			"strokeColor": "#f08c00",
			"backgroundColor": "transparent",
			"width": 32.22216796875,
			"height": 347.7777506510415,
			"seed": 1169526537,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0,
					2.2222493489582575
				],
				[
					1.111083984375,
					7.777750651041629
				],
				[
					3.333333333333485,
					15.555582682291629
				],
				[
					3.333333333333485,
					28.888916015625
				],
				[
					3.333333333333485,
					42.22224934895826
				],
				[
					3.333333333333485,
					57.77775065104163
				],
				[
					3.333333333333485,
					72.22224934895826
				],
				[
					3.333333333333485,
					83.33333333333326
				],
				[
					1.111083984375,
					92.22224934895826
				],
				[
					-1.111083984375,
					98.888916015625
				],
				[
					-1.111083984375,
					106.66666666666663
				],
				[
					-3.3333333333332575,
					111.111083984375
				],
				[
					-4.4444173177082575,
					114.44441731770826
				],
				[
					-5.555582682291515,
					114.44441731770826
				],
				[
					-7.777750651041515,
					114.44441731770826
				],
				[
					-15.555582682291515,
					113.33333333333326
				],
				[
					-20,
					108.888916015625
				],
				[
					-21.111083984375,
					106.66666666666663
				],
				[
					-23.333333333333258,
					103.33333333333326
				],
				[
					-23.333333333333258,
					101.111083984375
				],
				[
					-24.444417317708258,
					96.66666666666663
				],
				[
					-24.444417317708258,
					92.22224934895826
				],
				[
					-24.444417317708258,
					90
				],
				[
					-23.333333333333258,
					90
				],
				[
					-22.222249348958258,
					90
				],
				[
					-15.555582682291515,
					92.22224934895826
				],
				[
					-11.111083984375,
					97.77775065104163
				],
				[
					-8.888916015625,
					102.22224934895826
				],
				[
					-5.555582682291515,
					105.55558268229163
				],
				[
					-4.4444173177082575,
					113.33333333333326
				],
				[
					-3.3333333333332575,
					118.888916015625
				],
				[
					-2.2222493489582575,
					125.55558268229163
				],
				[
					-2.2222493489582575,
					134.44441731770826
				],
				[
					-2.2222493489582575,
					141.111083984375
				],
				[
					-2.2222493489582575,
					148.888916015625
				],
				[
					-2.2222493489582575,
					155.55558268229163
				],
				[
					-2.2222493489582575,
					163.33333333333326
				],
				[
					-2.2222493489582575,
					172.22224934895826
				],
				[
					-2.2222493489582575,
					183.33333333333326
				],
				[
					-3.3333333333332575,
					191.111083984375
				],
				[
					-4.4444173177082575,
					197.77775065104163
				],
				[
					-4.4444173177082575,
					204.44441731770826
				],
				[
					-4.4444173177082575,
					212.22224934895826
				],
				[
					-6.666666666666515,
					218.8889160156249
				],
				[
					-7.777750651041515,
					225.55558268229163
				],
				[
					-7.777750651041515,
					234.44441731770826
				],
				[
					-10,
					241.1110839843749
				],
				[
					-11.111083984375,
					249.9999999999999
				],
				[
					-11.111083984375,
					255.55558268229163
				],
				[
					-12.222249348958258,
					266.66666666666663
				],
				[
					-12.222249348958258,
					269.9999999999999
				],
				[
					-12.222249348958258,
					274.44441731770826
				],
				[
					-12.222249348958258,
					281.1110839843749
				],
				[
					-13.333333333333258,
					285.55558268229163
				],
				[
					-13.333333333333258,
					288.8889160156249
				],
				[
					-13.333333333333258,
					294.44441731770826
				],
				[
					-13.333333333333258,
					298.8889160156249
				],
				[
					-13.333333333333258,
					303.33333333333326
				],
				[
					-13.333333333333258,
					307.77775065104163
				],
				[
					-13.333333333333258,
					309.9999999999999
				],
				[
					-13.333333333333258,
					314.44441731770826
				],
				[
					-13.333333333333258,
					318.8889160156249
				],
				[
					-10,
					322.22224934895826
				],
				[
					-7.777750651041515,
					326.66666666666663
				],
				[
					-6.666666666666515,
					328.8889160156249
				],
				[
					-5.555582682291515,
					332.22224934895826
				],
				[
					-4.4444173177082575,
					334.44441731770826
				],
				[
					-3.3333333333332575,
					337.77775065104163
				],
				[
					-1.111083984375,
					341.1110839843749
				],
				[
					1.111083984375,
					343.33333333333326
				],
				[
					2.222249348958485,
					345.5555826822915
				],
				[
					3.333333333333485,
					345.5555826822915
				],
				[
					4.444417317708485,
					346.6666666666665
				],
				[
					5.5555826822917425,
					347.7777506510415
				],
				[
					6.6666666666667425,
					347.7777506510415
				],
				[
					7.7777506510417425,
					347.7777506510415
				],
				[
					7.7777506510417425,
					347.7777506510415
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "text",
			"version": 422,
			"versionNonce": 867460679,
			"isDeleted": false,
			"id": "c7hsFn1K",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6824.685822777129,
			"y": -85.93388928071806,
			"strokeColor": "#f08c00",
			"backgroundColor": "transparent",
			"width": 9.755996704101562,
			"height": 45,
			"seed": 404801001,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726370,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "1",
			"rawText": "1",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "1",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "text",
			"version": 433,
			"versionNonce": 1558267305,
			"isDeleted": false,
			"id": "rWdBpeLm",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6813.414330772735,
			"y": -235.48943127290556,
			"strokeColor": "#f08c00",
			"backgroundColor": "transparent",
			"width": 25.631988525390625,
			"height": 45,
			"seed": 700588233,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726370,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "2",
			"rawText": "2",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "2",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "text",
			"version": 449,
			"versionNonce": 1358824807,
			"isDeleted": false,
			"id": "1rSqbWm0",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6804.431407900827,
			"y": -4.822723916134578,
			"strokeColor": "#f08c00",
			"backgroundColor": "transparent",
			"width": 29.375991821289062,
			"height": 45,
			"seed": 2016747433,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726370,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "1.1",
			"rawText": "1.1",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "1.1",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "text",
			"version": 442,
			"versionNonce": 1239197833,
			"isDeleted": false,
			"id": "vqlbYsk3",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6798.048669151641,
			"y": 209.62161202136508,
			"strokeColor": "#f08c00",
			"backgroundColor": "transparent",
			"width": 45.251983642578125,
			"height": 45,
			"seed": 1101349513,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726370,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "1.2",
			"rawText": "1.2",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "1.2",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "arrow",
			"version": 808,
			"versionNonce": 404061257,
			"isDeleted": false,
			"id": "2IfpCidnZMC4UbUDvj1Ma",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8036.908950172094,
			"y": -781.0735511660123,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 2976.1901855468755,
			"height": 524.0157199181803,
			"seed": 1926105449,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1715236730699,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "dw1tHtsj",
				"focus": 0.8022475985478323,
				"gap": 9.35904348979841
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					2478.5710797991073,
					-18.140320459777822
				],
				[
					2976.1901855468755,
					505.8753994584024
				]
			]
		},
		{
			"type": "text",
			"version": 766,
			"versionNonce": 1328571045,
			"isDeleted": false,
			"id": "KCSYHi8C",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0.03559169220205671,
			"x": 10015.77823030602,
			"y": -814.3170394726772,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 182.34375,
			"height": 23,
			"seed": 1425732681,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Invoke and pass null",
			"rawText": "Invoke and pass null",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Invoke and pass null",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "rectangle",
			"version": 731,
			"versionNonce": 1148732971,
			"isDeleted": false,
			"id": "J9BKFCjYUHtMNog946eeL",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 4911.904594717292,
			"y": -936.5666094476665,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 7515.713500976561,
			"height": 2955.280220567162,
			"seed": 1736585545,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 586,
			"versionNonce": 1309065351,
			"isDeleted": false,
			"id": "Jyh0ilGE",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 4983.061930293655,
			"y": -897.7211149093064,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 1047.9957275390625,
			"height": 45,
			"seed": 345972777,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726371,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "POPUlATE INSPECTIONITEMS AND ARRAYS DATA GRID",
			"rawText": "POPUlATE INSPECTIONITEMS AND ARRAYS DATA GRID",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "POPUlATE INSPECTIONITEMS AND ARRAYS DATA GRID",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "text",
			"version": 290,
			"versionNonce": 187215721,
			"isDeleted": false,
			"id": "oYuRHNma",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5105.517639324725,
			"y": -550.8480998804788,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 264.33978271484375,
			"height": 25,
			"seed": 339296935,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726371,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "Selected PCBItem changed",
			"rawText": "Selected PCBItem changed",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Selected PCBItem changed",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "rectangle",
			"version": 446,
			"versionNonce": 307871077,
			"isDeleted": false,
			"id": "eR6QkmahUMZjD8dZ_FP94",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5067.627617162489,
			"y": -558.9530001797535,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 336.0528329170433,
			"height": 40.68177101828837,
			"seed": 426268103,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "4Zyks4FhkoVsfz0zc5uUl",
					"type": "arrow"
				}
			],
			"updated": 1712906619441,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 226,
			"versionNonce": 657913707,
			"isDeleted": false,
			"id": "owJcVV4a",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5263.806043425579,
			"y": 130.20170423143588,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 301.970703125,
			"height": 32.199999999999996,
			"seed": 1047496967,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 28,
			"fontFamily": 2,
			"text": "[Get condition data only]",
			"rawText": "[Get condition data only]",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "[Get condition data only]",
			"lineHeight": 1.15,
			"baseline": 25
		},
		{
			"type": "text",
			"version": 690,
			"versionNonce": 1164363973,
			"isDeleted": false,
			"id": "4T40NjuQ",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9944.310076964966,
			"y": 995.5463348264734,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 489.38623046875,
			"height": 82.31830973478105,
			"seed": 394605577,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 35.79056944990481,
			"fontFamily": 2,
			"text": "[CreateBoardInfoAsync create \ndata for array data grid]",
			"rawText": "[CreateBoardInfoAsync create \ndata for array data grid]",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "[CreateBoardInfoAsync create \ndata for array data grid]",
			"lineHeight": 1.15,
			"baseline": 74
		},
		{
			"type": "image",
			"version": 602,
			"versionNonce": 1071305227,
			"isDeleted": false,
			"id": "exW4q81Cgs4vKwIPqKanB",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10458.696520591195,
			"y": 978.8851061566593,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 865.1061473142905,
			"height": 894.8640983275585,
			"seed": 1752864327,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "UBow9MxWuWjvmv6487H78",
					"type": "arrow"
				}
			],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "e6eef5f2d9ebefcbc9e6e9d857f895d9290437a7",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "text",
			"version": 811,
			"versionNonce": 1568368677,
			"isDeleted": false,
			"id": "uYhWzbWO",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9944.310076964966,
			"y": 1144.7676115378963,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 483.39208984375,
			"height": 82.31830973478105,
			"seed": 41803527,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 35.79056944990481,
			"fontFamily": 2,
			"text": "[It also need condition data in \nbyte format for board size info]",
			"rawText": "[It also need condition data in \nbyte format for board size info]",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "[It also need condition data in \nbyte format for board size info]",
			"lineHeight": 1.15,
			"baseline": 74
		},
		{
			"type": "text",
			"version": 878,
			"versionNonce": 1483364523,
			"isDeleted": false,
			"id": "UmTRpWra",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9942.29385488637,
			"y": 1290.2747377467488,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 517.1199340820312,
			"height": 41.15915486739053,
			"seed": 799756231,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 35.79056944990481,
			"fontFamily": 2,
			"text": "[This func is in CCIResultParser]",
			"rawText": "[This func is in CCIResultParser]",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "[This func is in CCIResultParser]",
			"lineHeight": 1.15,
			"baseline": 33
		},
		{
			"type": "text",
			"version": 737,
			"versionNonce": 2089098117,
			"isDeleted": false,
			"id": "quoYd7cs",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5260.970240539032,
			"y": 68.83516219401679,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 513.2054443359375,
			"height": 41.15915486739053,
			"seed": 933393417,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619441,
			"link": null,
			"locked": false,
			"fontSize": 35.79056944990481,
			"fontFamily": 2,
			"text": "[This func is in ResultDbReader]",
			"rawText": "[This func is in ResultDbReader]",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "[This func is in ResultDbReader]",
			"lineHeight": 1.15,
			"baseline": 33
		},
		{
			"type": "text",
			"version": 901,
			"versionNonce": 478877607,
			"isDeleted": false,
			"id": "oRS1TJuo",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8385.256580909358,
			"y": 1198.159377663229,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 291.92388916015625,
			"height": 45,
			"seed": 426628969,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726372,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "ResultDBReader",
			"rawText": "ResultDBReader",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "ResultDBReader",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "rectangle",
			"version": 879,
			"versionNonce": 717316837,
			"isDeleted": false,
			"id": "1dT_3uoJogyMriSk4X6Dg",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8342.756711475406,
			"y": 1167.8816168396615,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 363.88895670572924,
			"height": 105.55552164713549,
			"seed": 1920008777,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "9x69R7pWDlHnPUCh52Gk4",
					"type": "arrow"
				},
				{
					"id": "6WlcC680rTFn2z9Dr6SV-",
					"type": "arrow"
				}
			],
			"updated": 1712906619441,
			"link": null,
			"locked": false
		},
		{
			"type": "arrow",
			"version": 2676,
			"versionNonce": 1722182123,
			"isDeleted": false,
			"id": "6WlcC680rTFn2z9Dr6SV-",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 1.2193395021038915,
			"x": 8206.15504220449,
			"y": 1048.2289259905083,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 18.432017261037515,
			"height": 700.5940720337692,
			"seed": 1535889705,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "1dT_3uoJogyMriSk4X6Dg",
				"focus": -0.5888766541795282,
				"gap": 12.944337754504318
			},
			"endBinding": {
				"elementId": "LODHKiMf",
				"focus": 0.8368101967196419,
				"gap": 10.824709892372084
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-18.432017261037515,
					700.5940720337692
				]
			]
		},
		{
			"type": "text",
			"version": 1049,
			"versionNonce": 386980425,
			"isDeleted": false,
			"id": "LODHKiMf",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 6.278915113813003,
			"x": 7468.528393906811,
			"y": 1492.8823695505175,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 385.61566162109375,
			"height": 35,
			"seed": 975815689,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "6WlcC680rTFn2z9Dr6SV-",
					"type": "arrow"
				},
				{
					"id": "fFeH74LlPgut0DGu_gutF",
					"type": "arrow"
				},
				{
					"id": "KMWZo-0qCnJdo42-22BqB",
					"type": "arrow"
				}
			],
			"updated": 1715236726372,
			"link": null,
			"locked": false,
			"fontSize": 28,
			"fontFamily": 1,
			"text": "CreateInspectionItemsAsync",
			"rawText": "CreateInspectionItemsAsync",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "CreateInspectionItemsAsync",
			"lineHeight": 1.25,
			"baseline": 25
		},
		{
			"type": "image",
			"version": 811,
			"versionNonce": 1974989963,
			"isDeleted": false,
			"id": "yOCpeCn1X3NnDcEDgSdcv",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6933.256570997667,
			"y": 1115.1577684444728,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 993.9999999999999,
			"height": 203,
			"seed": 977689321,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "9x69R7pWDlHnPUCh52Gk4",
					"type": "arrow"
				},
				{
					"id": "bxOBkKKZot3M9is-k2wHo",
					"type": "arrow"
				},
				{
					"id": "huqsBeTB_ACadsA_DA66T",
					"type": "arrow"
				}
			],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "99546c1915d73087b33992c90dc4cdc16716f674",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 2129,
			"versionNonce": 1498918309,
			"isDeleted": false,
			"id": "9x69R7pWDlHnPUCh52Gk4",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8332.03436233881,
			"y": 1227.2134325069728,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 390,
			"height": 1.6666259765625,
			"seed": 2087523785,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "1dT_3uoJogyMriSk4X6Dg",
				"focus": -0.1377529335955515,
				"gap": 10.722349136593948
			},
			"endBinding": {
				"elementId": "yOCpeCn1X3NnDcEDgSdcv",
				"focus": 0.06467627846055488,
				"gap": 14.777791341143711
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-390,
					-1.6666259765625
				]
			]
		},
		{
			"type": "text",
			"version": 892,
			"versionNonce": 1119324871,
			"isDeleted": false,
			"id": "F6il80c9",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8103.539145969182,
			"y": 1176.713463024551,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 110.40396118164062,
			"height": 35,
			"seed": 62485673,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726373,
			"link": null,
			"locked": false,
			"fontSize": 28,
			"fontFamily": 1,
			"text": "Creates",
			"rawText": "Creates",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Creates",
			"lineHeight": 1.25,
			"baseline": 24
		},
		{
			"type": "image",
			"version": 788,
			"versionNonce": 264024325,
			"isDeleted": false,
			"id": "IMrz2luLmWgwMp3xOBf62",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5818.034260613552,
			"y": 1048.7371199604704,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 898,
			"height": 323,
			"seed": 1675032457,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "bxOBkKKZot3M9is-k2wHo",
					"type": "arrow"
				},
				{
					"id": "KMWZo-0qCnJdo42-22BqB",
					"type": "arrow"
				},
				{
					"id": "ZxA924oD-zrmPxnOt5p_S",
					"type": "arrow"
				}
			],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "c10555ce38240187e07582a50a935848cb22309e",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 2257,
			"versionNonce": 1550224843,
			"isDeleted": false,
			"id": "bxOBkKKZot3M9is-k2wHo",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6919.004354801481,
			"y": 1201.483351468144,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 197.92426140437453,
			"height": 1.5617479090312827,
			"seed": 788602473,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "yOCpeCn1X3NnDcEDgSdcv",
				"focus": 0.10567386649071944,
				"gap": 14.252216196186055
			},
			"endBinding": {
				"elementId": "IMrz2luLmWgwMp3xOBf62",
				"focus": -0.084209780255046,
				"gap": 5.045832783554943
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-197.92426140437453,
					-1.5617479090312827
				]
			]
		},
		{
			"type": "text",
			"version": 957,
			"versionNonce": 242264361,
			"isDeleted": false,
			"id": "3tgX9r7t",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6778.12824417962,
			"y": 1144.4216674325326,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 115.52793884277344,
			"height": 35,
			"seed": 604203337,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "bxOBkKKZot3M9is-k2wHo",
					"type": "arrow"
				}
			],
			"updated": 1715236726373,
			"link": null,
			"locked": false,
			"fontSize": 28,
			"fontFamily": 1,
			"text": "Contains",
			"rawText": "Contains",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Contains",
			"lineHeight": 1.25,
			"baseline": 24
		},
		{
			"type": "arrow",
			"version": 2799,
			"versionNonce": 646473454,
			"isDeleted": false,
			"id": "fFeH74LlPgut0DGu_gutF",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 1.2193395021038915,
			"x": 7240.882892536883,
			"y": 1420.5996967046149,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 21.702326258280664,
			"height": 373.53032951773366,
			"seed": 473520169,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1713163607054,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "BX0dj0aHaFfo43AEcnic5",
				"focus": 1.083887875170369,
				"gap": 29.977327707686754
			},
			"endBinding": {
				"elementId": "_V0w3ebtJl12foM6iyveV",
				"focus": 0.8799589839199771,
				"gap": 12.738611744115133
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					21.702326258280664,
					373.53032951773366
				]
			]
		},
		{
			"type": "image",
			"version": 800,
			"versionNonce": 606991301,
			"isDeleted": false,
			"id": "wipclUaFh8LjbXoiBk24a",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7079.771999220432,
			"y": 835.8692902702223,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 689,
			"height": 137,
			"seed": 965634825,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "huqsBeTB_ACadsA_DA66T",
					"type": "arrow"
				}
			],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "b098342fd39a640720d08ead3fd39295bd7c34ce",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 2099,
			"versionNonce": 1957680907,
			"isDeleted": false,
			"id": "huqsBeTB_ACadsA_DA66T",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7422.271755079807,
			"y": 987.3691071647536,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 0,
			"height": 118.3333740234375,
			"seed": 1417552361,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "wipclUaFh8LjbXoiBk24a",
				"focus": 0.005806223920537011,
				"gap": 14.49981689453125
			},
			"endBinding": {
				"elementId": "yOCpeCn1X3NnDcEDgSdcv",
				"focus": -0.016066028003742523,
				"gap": 9.455287256281736
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					0,
					118.3333740234375
				]
			]
		},
		{
			"type": "text",
			"version": 998,
			"versionNonce": 1294506471,
			"isDeleted": false,
			"id": "lVWIGlPs",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7455.50778565842,
			"y": 1016.5357941764723,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 103.93594360351562,
			"height": 35,
			"seed": 705952969,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726373,
			"link": null,
			"locked": false,
			"fontSize": 28,
			"fontFamily": 1,
			"text": "Keys of",
			"rawText": "Keys of",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Keys of",
			"lineHeight": 1.25,
			"baseline": 24
		},
		{
			"type": "text",
			"version": 1104,
			"versionNonce": 374783403,
			"isDeleted": false,
			"id": "4UpwUazF",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6458.31099051275,
			"y": 1722.4246695019938,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 541.34765625,
			"height": 23,
			"seed": 1240169385,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "[Get all overview tables needed from specify ResultDBName]",
			"rawText": "[Get all overview tables needed from specify ResultDBName]",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "[Get all overview tables needed from specify ResultDBName]",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "image",
			"version": 808,
			"versionNonce": 767395461,
			"isDeleted": false,
			"id": "_V0w3ebtJl12foM6iyveV",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6371.604762892309,
			"y": 1658.024158720531,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 695.7777506510417,
			"height": 50.511472695525526,
			"seed": 1144004233,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "fFeH74LlPgut0DGu_gutF",
					"type": "arrow"
				},
				{
					"id": "KMWZo-0qCnJdo42-22BqB",
					"type": "arrow"
				}
			],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "71cbfdb3dffabfcc528868b84c0e6ad55409afbf",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 2154,
			"versionNonce": 1000853579,
			"isDeleted": false,
			"id": "KMWZo-0qCnJdo42-22BqB",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7458.604762892309,
			"y": 1513.7021760124117,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 1201.1110839843748,
			"height": 127.77775065104169,
			"seed": 396191081,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "LODHKiMf",
				"focus": -0.6367188668265094,
				"gap": 9.935605568954088
			},
			"endBinding": {
				"elementId": "IMrz2luLmWgwMp3xOBf62",
				"focus": 0.8443908914593676,
				"gap": 14.187305400899504
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-1201.1110839843748,
					-127.77775065104169
				]
			]
		},
		{
			"type": "image",
			"version": 801,
			"versionNonce": 263482853,
			"isDeleted": false,
			"id": "BX0dj0aHaFfo43AEcnic5",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7469.646527469961,
			"y": 1549.7291110050446,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 1108,
			"height": 385,
			"seed": 1602146377,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "fFeH74LlPgut0DGu_gutF",
					"type": "arrow"
				}
			],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "c0f232272baf8178f58963be283c03071cb11777",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "text",
			"version": 1099,
			"versionNonce": 1790155785,
			"isDeleted": false,
			"id": "qg0ciWZ2",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0.12187160175879974,
			"x": 6894.492941090483,
			"y": 1436.0542472577627,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 366.93988037109375,
			"height": 35,
			"seed": 1967057129,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726374,
			"link": null,
			"locked": false,
			"fontSize": 28,
			"fontFamily": 1,
			"text": "Uses to create InspItems",
			"rawText": "Uses to create InspItems",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Uses to create InspItems",
			"lineHeight": 1.25,
			"baseline": 24
		},
		{
			"type": "text",
			"version": 980,
			"versionNonce": 353855813,
			"isDeleted": false,
			"id": "qu7nTP8h",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5919.422497037635,
			"y": 936.8921977046921,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 690.443359375,
			"height": 96.6,
			"seed": 2051189865,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"fontSize": 28,
			"fontFamily": 2,
			"text": "Each one of this get ORM from DB then convert to DTO\n    + ORM data type is the item table name\n    + DTO data type is InspectionItem",
			"rawText": "Each one of this get ORM from DB then convert to DTO\n    + ORM data type is the item table name\n    + DTO data type is InspectionItem",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Each one of this get ORM from DB then convert to DTO\n    + ORM data type is the item table name\n    + DTO data type is InspectionItem",
			"lineHeight": 1.15,
			"baseline": 89
		},
		{
			"type": "arrow",
			"version": 1140,
			"versionNonce": 2036496779,
			"isDeleted": false,
			"id": "ZxA924oD-zrmPxnOt5p_S",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5803.6712898256965,
			"y": 1203.2656512436265,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 1937.878639914772,
			"height": 730.3029563210227,
			"seed": 300175783,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "IMrz2luLmWgwMp3xOBf62",
				"focus": 0.946388172513551,
				"gap": 14.362970787855375
			},
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-192.42420543323806,
					730.3029563210227
				],
				[
					1745.454434481534,
					677.2727272727273
				]
			]
		},
		{
			"type": "text",
			"version": 1274,
			"versionNonce": 15385863,
			"isDeleted": false,
			"id": "HCsguqsc",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 6.214006030583311,
			"x": 6544.399226810671,
			"y": 1882.8400808424663,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 451.05181884765625,
			"height": 35,
			"seed": 863350697,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726374,
			"link": null,
			"locked": false,
			"fontSize": 28,
			"fontFamily": 1,
			"text": "Add to a list of InpsItems here",
			"rawText": "Add to a list of InpsItems here",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Add to a list of InpsItems here",
			"lineHeight": 1.25,
			"baseline": 24
		},
		{
			"type": "text",
			"version": 736,
			"versionNonce": 1636896811,
			"isDeleted": false,
			"id": "npWbc3c4",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6857.440417697603,
			"y": 881.4803514275795,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 95.32154846191406,
			"height": 179.4611310951241,
			"seed": 1326293223,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"fontSize": 156.05315747402096,
			"fontFamily": 2,
			"text": "Z",
			"rawText": "Z",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Z",
			"lineHeight": 1.15,
			"baseline": 143
		},
		{
			"type": "text",
			"version": 644,
			"versionNonce": 132619269,
			"isDeleted": false,
			"id": "tOi2r79E",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 11355.574065626988,
			"y": 1105.287131831202,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 104.08412170410156,
			"height": 179.4611310951241,
			"seed": 1004909863,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"fontSize": 156.05315747402096,
			"fontFamily": 2,
			"text": "Y",
			"rawText": "Y",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Y",
			"lineHeight": 1.15,
			"baseline": 143
		},
		{
			"type": "text",
			"version": 453,
			"versionNonce": 15979211,
			"isDeleted": false,
			"id": "jOBlFePc",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5101.391189574347,
			"y": 270.43914544426354,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 104.08412170410156,
			"height": 179.4611310951241,
			"seed": 622977321,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"fontSize": 156.05315747402096,
			"fontFamily": 2,
			"text": "X",
			"rawText": "X",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "X",
			"lineHeight": 1.15,
			"baseline": 143
		},
		{
			"type": "text",
			"version": 448,
			"versionNonce": 385864549,
			"isDeleted": false,
			"id": "BGMSNTTT",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9278.092697715185,
			"y": -261.31769626207074,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 38.567688890880405,
			"height": 66.49814552654772,
			"seed": 133144617,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"fontSize": 57.82447437091106,
			"fontFamily": 2,
			"text": "X",
			"rawText": "X",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "X",
			"lineHeight": 1.15,
			"baseline": 53
		},
		{
			"type": "text",
			"version": 463,
			"versionNonce": 941965675,
			"isDeleted": false,
			"id": "0wHshjOE",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10268.106724175455,
			"y": -344.94435037645746,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 38.567688890880405,
			"height": 66.49814552654772,
			"seed": 587530471,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"fontSize": 57.82447437091106,
			"fontFamily": 2,
			"text": "Y",
			"rawText": "Y",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Y",
			"lineHeight": 1.15,
			"baseline": 53
		},
		{
			"type": "text",
			"version": 549,
			"versionNonce": 953818821,
			"isDeleted": false,
			"id": "dPFZRATQ",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9433.090778036216,
			"y": 110.88067231135784,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 35.32077482603392,
			"height": 66.49814552654772,
			"seed": 1789333641,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"fontSize": 57.82447437091106,
			"fontFamily": 2,
			"text": "Z",
			"rawText": "Z",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Z",
			"lineHeight": 1.15,
			"baseline": 53
		},
		{
			"type": "text",
			"version": 274,
			"versionNonce": 1886379019,
			"isDeleted": false,
			"id": "VYORq4dF",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10692.913545840349,
			"y": 377.89127623078525,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 1200.69140625,
			"height": 41.4,
			"seed": 789424583,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 2,
			"text": "[Handle when PCBItems changed or when InspectionItemResults changed]",
			"rawText": "[Handle when PCBItems changed or when InspectionItemResults changed]",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "[Handle when PCBItems changed or when InspectionItemResults changed]",
			"lineHeight": 1.15,
			"baseline": 33
		},
		{
			"type": "text",
			"version": 799,
			"versionNonce": 1809134313,
			"isDeleted": false,
			"id": "L7edmt0c",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1645.310434502393,
			"y": 357.38032577825766,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 492.1439113205683,
			"height": 75.86387011059414,
			"seed": 1472810759,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726375,
			"link": null,
			"locked": false,
			"fontSize": 60.69109608847531,
			"fontFamily": 1,
			"text": "ResultDBReader",
			"rawText": "ResultDBReader",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "ResultDBReader",
			"lineHeight": 1.25,
			"baseline": 54
		},
		{
			"type": "rectangle",
			"version": 772,
			"versionNonce": 97395371,
			"isDeleted": false,
			"id": "qvobAmt-oKVABhN8LNFiG",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1584.6487840937552,
			"y": 306.3361454591736,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 613.4672121378458,
			"height": 177.95223074876213,
			"seed": 469227047,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 858,
			"versionNonce": 1299876903,
			"isDeleted": false,
			"id": "eX5Bni69",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2450.1867926442064,
			"y": 357.38032577825766,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 502.20965576171875,
			"height": 75.86387011059414,
			"seed": 1661499719,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726376,
			"link": null,
			"locked": false,
			"fontSize": 60.69109608847531,
			"fontFamily": 1,
			"text": "CCIResultParser",
			"rawText": "CCIResultParser",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "CCIResultParser",
			"lineHeight": 1.25,
			"baseline": 53
		},
		{
			"type": "rectangle",
			"version": 789,
			"versionNonce": 1894421835,
			"isDeleted": false,
			"id": "UzA9z71SaLlZZPKRaA07K",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2394.5580144561427,
			"y": 306.3361454591736,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 613.4672121378458,
			"height": 177.95223074876213,
			"seed": 1113045095,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 832,
			"versionNonce": 1180757449,
			"isDeleted": false,
			"id": "ZYb0QUD0",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3363.6974588355924,
			"y": 357.38032577825766,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 430.71685791015625,
			"height": 75.86387011059414,
			"seed": 1873941383,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726376,
			"link": null,
			"locked": false,
			"fontSize": 60.69109608847531,
			"fontFamily": 1,
			"text": "ResultProvider",
			"rawText": "ResultProvider",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "ResultProvider",
			"lineHeight": 1.25,
			"baseline": 53
		},
		{
			"type": "rectangle",
			"version": 788,
			"versionNonce": 824161259,
			"isDeleted": false,
			"id": "OMUfdISYdBLILfvKWAKxO",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3272.3222817217475,
			"y": 306.3361454591736,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 613.4672121378458,
			"height": 177.95223074876213,
			"seed": 861145767,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 739,
			"versionNonce": 953553989,
			"isDeleted": false,
			"id": "yNRwP3su",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1528.2572070572096,
			"y": 548.246409281023,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 736.1939697265625,
			"height": 164.6366194695621,
			"seed": 434773447,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"fontSize": 35.79056944990481,
			"fontFamily": 2,
			"text": "Help create data needed on the left data grids:\n    + PCBItems\n    + InspectionItems\n    + InspectionItemResults",
			"rawText": "Help create data needed on the left data grids:\n    + PCBItems\n    + InspectionItems\n    + InspectionItemResults",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Help create data needed on the left data grids:\n    + PCBItems\n    + InspectionItems\n    + InspectionItemResults",
			"lineHeight": 1.15,
			"baseline": 156
		},
		{
			"type": "text",
			"version": 885,
			"versionNonce": 1987575435,
			"isDeleted": false,
			"id": "gaf0PtvF",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2333.1946356617846,
			"y": 549.8606106145446,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 736.1939697265625,
			"height": 205.79577433695263,
			"seed": 1720765671,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"fontSize": 35.79056944990481,
			"fontFamily": 2,
			"text": "Help create data needed on the left data grids:\n    + Arrays\n    + Components\n    + ComponentDrawInfos\n    + InspectionProperties",
			"rawText": "Help create data needed on the left data grids:\n    + Arrays\n    + Components\n    + ComponentDrawInfos\n    + InspectionProperties",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Help create data needed on the left data grids:\n    + Arrays\n    + Components\n    + ComponentDrawInfos\n    + InspectionProperties",
			"lineHeight": 1.15,
			"baseline": 197
		},
		{
			"type": "text",
			"version": 1169,
			"versionNonce": 1274364837,
			"isDeleted": false,
			"id": "IgwFQYe1",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3141.860988973396,
			"y": 547.831984662474,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 907.157470703125,
			"height": 164.6366194695621,
			"seed": 1154783239,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"fontSize": 35.79056944990481,
			"fontFamily": 2,
			"text": "Convert IMeasureValue inside an InspectionItemResult\nto corresponding defect data.\nFunctions in this class usually get called when the control\nneed data related to multiple different grids.",
			"rawText": "Convert IMeasureValue inside an InspectionItemResult\nto corresponding defect data.\nFunctions in this class usually get called when the control\nneed data related to multiple different grids.",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Convert IMeasureValue inside an InspectionItemResult\nto corresponding defect data.\nFunctions in this class usually get called when the control\nneed data related to multiple different grids.",
			"lineHeight": 1.15,
			"baseline": 156
		},
		{
			"type": "rectangle",
			"version": 791,
			"versionNonce": 195949867,
			"isDeleted": false,
			"id": "29vkAdRYTTfg-7hOE6gig",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 4911.752588972592,
			"y": 2124.6825296307616,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 7515.713500976561,
			"height": 2955.280220567162,
			"seed": 304229095,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [
				{
					"id": "6WlcC680rTFn2z9Dr6SV-",
					"type": "arrow"
				},
				{
					"id": "hjWu72m65d1PDe6_Fhmoh",
					"type": "arrow"
				}
			],
			"updated": 1712906619442,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 691,
			"versionNonce": 2126802759,
			"isDeleted": false,
			"id": "wJ1vQ4rm",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 4982.909924548955,
			"y": 2163.5280241691216,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 602.0638427734375,
			"height": 45,
			"seed": 1921660423,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726377,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "POPULATE OTHER DATA GRIDS",
			"rawText": "POPULATE OTHER DATA GRIDS",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "POPULATE OTHER DATA GRIDS",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "line",
			"version": 90,
			"versionNonce": 2889675,
			"isDeleted": false,
			"id": "6Yr6g84X2ivbI6dCCV0Eh",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "dashed",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8661.327776067474,
			"y": 2138.3506015953344,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 16.66748046875,
			"height": 2958.3331298828125,
			"seed": 641901959,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": null,
			"points": [
				[
					0,
					0
				],
				[
					16.66748046875,
					2958.3331298828125
				]
			]
		},
		{
			"type": "text",
			"version": 639,
			"versionNonce": 292703401,
			"isDeleted": false,
			"id": "CfPR3FQk",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 4988.850184057018,
			"y": 2255.7664394433323,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 718.7037963867188,
			"height": 45,
			"seed": 800487367,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726378,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "POPULATE INSPECTIONITEMRESULTS",
			"rawText": "POPULATE INSPECTIONITEMRESULTS",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "POPULATE INSPECTIONITEMRESULTS",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "text",
			"version": 753,
			"versionNonce": 790949479,
			"isDeleted": false,
			"id": "6ZasjKB3",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8729.829564346732,
			"y": 2255.7664394433323,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 1488.8155517578125,
			"height": 45,
			"seed": 833747497,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726378,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "POPULATE COMPONENTS, COMPONENT DRAW INFOS, INSPECTION PROPERTIES",
			"rawText": "POPULATE COMPONENTS, COMPONENT DRAW INFOS, INSPECTION PROPERTIES",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "POPULATE COMPONENTS, COMPONENT DRAW INFOS, INSPECTION PROPERTIES",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "image",
			"version": 254,
			"versionNonce": 241912261,
			"isDeleted": false,
			"id": "BMZLqBNf5vfxAmhMODTUS",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5076.508921124865,
			"y": 2630.046048497999,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 816,
			"height": 691,
			"seed": 624524743,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "hjWu72m65d1PDe6_Fhmoh",
					"type": "arrow"
				},
				{
					"id": "oLeLxbGKbc5L_rDAGOCTn",
					"type": "arrow"
				}
			],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "be07c77f5fb5d9a74157baf53abab76d7db8f3f2",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 720,
			"versionNonce": 1478156555,
			"isDeleted": false,
			"id": "hjWu72m65d1PDe6_Fhmoh",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6101.816255717355,
			"y": 1369.9660971383237,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 268.6505417596718,
			"height": 1228.5713413783483,
			"seed": 284437735,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "BMZLqBNf5vfxAmhMODTUS",
				"focus": 0.43680818231978247,
				"gap": 31.508609981327027
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-20.238037109375,
					459.1269211542037
				],
				[
					-268.6505417596718,
					1228.5713413783483
				]
			]
		},
		{
			"type": "text",
			"version": 614,
			"versionNonce": 1950861605,
			"isDeleted": false,
			"id": "mymuNVO7",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5094.793930950436,
			"y": 2504.4941790736134,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 690.443359375,
			"height": 96.6,
			"seed": 1807572775,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "hjWu72m65d1PDe6_Fhmoh",
					"type": "arrow"
				}
			],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"fontSize": 28,
			"fontFamily": 2,
			"text": "Each one of this get ORM from DB then convert to DTO\n    + ORM data type is the item result table name\n    + DTO data type is InspectionItemResult",
			"rawText": "Each one of this get ORM from DB then convert to DTO\n    + ORM data type is the item result table name\n    + DTO data type is InspectionItemResult",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Each one of this get ORM from DB then convert to DTO\n    + ORM data type is the item result table name\n    + DTO data type is InspectionItemResult",
			"lineHeight": 1.15,
			"baseline": 89
		},
		{
			"type": "text",
			"version": 1145,
			"versionNonce": 1958006665,
			"isDeleted": false,
			"id": "wvnqofMR",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5952.496843180632,
			"y": 2307.8501720063687,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 355.31982421875,
			"height": 35,
			"seed": 23920201,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726379,
			"link": null,
			"locked": false,
			"fontSize": 28,
			"fontFamily": 1,
			"text": "Called these in their body",
			"rawText": "Called these in their body",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Called these in their body",
			"lineHeight": 1.25,
			"baseline": 24
		},
		{
			"type": "image",
			"version": 104,
			"versionNonce": 488463493,
			"isDeleted": false,
			"id": "gBe2DsrHDtgQZOmRi1tLm",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5148.79950491078,
			"y": 3484.007991705387,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 668.0200920482673,
			"height": 700.078125,
			"seed": 1607060455,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "oLeLxbGKbc5L_rDAGOCTn",
					"type": "arrow"
				},
				{
					"id": "xgoeeykfLvaYJiUcq4_tQ",
					"type": "arrow"
				}
			],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "3e4c612bba5d3e49da99528062a9623fb76293aa",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 50,
			"versionNonce": 1398299211,
			"isDeleted": false,
			"id": "oLeLxbGKbc5L_rDAGOCTn",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5469.059550934914,
			"y": 3332.796977911442,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 2.083282470703125,
			"height": 135.41671752929688,
			"seed": 2103449255,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "BMZLqBNf5vfxAmhMODTUS",
				"focus": 0.02408171056991325,
				"gap": 11.750929413442918
			},
			"endBinding": {
				"elementId": "gBe2DsrHDtgQZOmRi1tLm",
				"focus": -0.06323410869265787,
				"gap": 15.794296264648438
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-2.083282470703125,
					135.41671752929688
				]
			]
		},
		{
			"type": "text",
			"version": 1197,
			"versionNonce": 2036894087,
			"isDeleted": false,
			"id": "K3WTSsew",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5487.399791413429,
			"y": 3379.8804129700356,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 139.99993896484375,
			"height": 35,
			"seed": 1608991943,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726379,
			"link": null,
			"locked": false,
			"fontSize": 28,
			"fontFamily": 1,
			"text": "Use these",
			"rawText": "Use these",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Use these",
			"lineHeight": 1.25,
			"baseline": 24
		},
		{
			"type": "image",
			"version": 253,
			"versionNonce": 1804392683,
			"isDeleted": false,
			"id": "x9hND6Dd0TzBUagnWL19G",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5175.392833405617,
			"y": 4338.2136191467935,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 619,
			"height": 630,
			"seed": 2118544841,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "xgoeeykfLvaYJiUcq4_tQ",
					"type": "arrow"
				}
			],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "02599db85914c06fc5008454440cd927947300af",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 69,
			"versionNonce": 49921861,
			"isDeleted": false,
			"id": "xgoeeykfLvaYJiUcq4_tQ",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5481.347306119311,
			"y": 4190.782215510606,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 2.083282470703125,
			"height": 135.41671752929688,
			"seed": 610418759,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "gBe2DsrHDtgQZOmRi1tLm",
				"focus": -0.01186184491573183,
				"gap": 6.696098805218753
			},
			"endBinding": {
				"elementId": "x9hND6Dd0TzBUagnWL19G",
				"focus": -0.03391065599075357,
				"gap": 12.014686106890622
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-2.083282470703125,
					135.41671752929688
				]
			]
		},
		{
			"type": "text",
			"version": 1248,
			"versionNonce": 375649897,
			"isDeleted": false,
			"id": "q0pvSMMw",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 5506.976909333351,
			"y": 4229.047206793278,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 201.54391479492188,
			"height": 35,
			"seed": 2090583335,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726379,
			"link": null,
			"locked": false,
			"fontSize": 28,
			"fontFamily": 1,
			"text": "to create this",
			"rawText": "to create this",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "to create this",
			"lineHeight": 1.25,
			"baseline": 24
		},
		{
			"type": "image",
			"version": 191,
			"versionNonce": 174821029,
			"isDeleted": false,
			"id": "GRTIOIQ4qalP0Wg3GJAXR",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6248.311213324605,
			"y": 2761.567904250728,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 863.128054755956,
			"height": 929.2911809044962,
			"seed": 34403399,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "ec98425227d056bee29653ac021dfccfee948d0a",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "text",
			"version": 1281,
			"versionNonce": 1791738023,
			"isDeleted": false,
			"id": "AsyHPcks",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6257.352877176542,
			"y": 2665.4163888349917,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 350.20037841796875,
			"height": 77.85714285714253,
			"seed": 452602729,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726379,
			"link": null,
			"locked": false,
			"fontSize": 62.28571428571403,
			"fontFamily": 1,
			"text": "Sample flow",
			"rawText": "Sample flow",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Sample flow",
			"lineHeight": 1.25,
			"baseline": 54
		},
		{
			"type": "image",
			"version": 449,
			"versionNonce": 363622917,
			"isDeleted": false,
			"id": "Oufg2NphyAVSpheUtU3Md",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -4697.68305529505,
			"y": 5039.142529675641,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 581.9063698849104,
			"height": 700.0781249999999,
			"seed": 650908935,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "99aee55d41b75c790cd894616689044b3f29c968",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 586,
			"versionNonce": 1057971403,
			"isDeleted": false,
			"id": "EcIx1ETw8rGg27ymc-dHg",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -43.4874702279385,
			"y": 4368.861623604693,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 413,
			"height": 111,
			"seed": 1253170215,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "-qrdiWSzcu7tOZwdvoxXq",
					"type": "arrow"
				},
				{
					"id": "MQ0Om5Ll8pbLfXKv4JO3-",
					"type": "arrow"
				},
				{
					"id": "Dg1Ycu0GGmp5hmBBtd9_V",
					"type": "arrow"
				},
				{
					"id": "4J04GG3GDOI0WY8edJitg",
					"type": "arrow"
				},
				{
					"id": "D8Dk2rMDZCwxZQ0AoQFbJ",
					"type": "arrow"
				},
				{
					"id": "VVzO4wcazVpq0n6IagEzW",
					"type": "arrow"
				},
				{
					"id": "P88whtN-r4Ozh6v_C--is",
					"type": "arrow"
				},
				{
					"id": "3jtpU_OUwPvr9_3FKp2f9",
					"type": "arrow"
				},
				{
					"id": "_vw7emRXN8UAbM9S-1-7o",
					"type": "arrow"
				},
				{
					"id": "xxI0L37132pHM9Mq1RvyF",
					"type": "arrow"
				},
				{
					"id": "OFsC6K3kEf0I7TEoQVpSf",
					"type": "arrow"
				},
				{
					"id": "fE19oYbgfLP9PrZJAavQp",
					"type": "arrow"
				},
				{
					"id": "hFlXN8gJTGWtKq6WAxge5",
					"type": "arrow"
				},
				{
					"id": "HzcK-jOZ30e1ayy6gWFeB",
					"type": "arrow"
				},
				{
					"id": "GpvxkvTarPpf-cGuPq7HQ",
					"type": "arrow"
				},
				{
					"id": "Cy-lDSEW3QcKKxTZgzKAK",
					"type": "arrow"
				},
				{
					"id": "fqDJ8bRvfbnVSHCKLmiVE",
					"type": "arrow"
				},
				{
					"id": "tayEKHOA0faSQBcSHcylK",
					"type": "arrow"
				}
			],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "5a37291430c63e6e34b0bcb73e4bcda3b6548290",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 406,
			"versionNonce": 28961125,
			"isDeleted": false,
			"id": "J5Qtg6mypiCbRRVC6HtEy",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2392.5127433951084,
			"y": 3446.6947687462944,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 619,
			"height": 265,
			"seed": 2022817607,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "-qrdiWSzcu7tOZwdvoxXq",
					"type": "arrow"
				},
				{
					"id": "vIsbvCunYqyJfqY3MwmuG",
					"type": "arrow"
				},
				{
					"id": "6nBzTKuf6FooP0sA11hOE",
					"type": "arrow"
				},
				{
					"id": "sjTLyI7mSFGyp1-IriJDM",
					"type": "arrow"
				},
				{
					"id": "KlrXPdAsXI6As2defktiq",
					"type": "arrow"
				}
			],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "d535e0549394692c4be4a077fc1da946e7e87cd4",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 772,
			"versionNonce": 2118093675,
			"isDeleted": false,
			"id": "Wh5YbyQ1rV_pHaeQ3IsIl",
			"fillStyle": "solid",
			"strokeWidth": 0.5,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -4023.9667051254555,
			"y": 5049.742804722713,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 527.5760731293358,
			"height": 1210.2401257612585,
			"seed": 583548519,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "2805bbe0fc2d5da322022f58df9e1fe55179e931",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "line",
			"version": 865,
			"versionNonce": 803304645,
			"isDeleted": false,
			"id": "QMrHLv5CpXDZ8Clzrp0V7",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3755.8679822957974,
			"y": 5059.017873824778,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 3752.2023794991655,
			"height": 617.5464024619444,
			"seed": 1008851335,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": null,
			"points": [
				[
					0,
					0
				],
				[
					2701.6470651778,
					-585.7011185903395
				],
				[
					3752.2023794991655,
					-617.5464024619444
				]
			]
		},
		{
			"type": "text",
			"version": 481,
			"versionNonce": 1229668681,
			"isDeleted": false,
			"id": "qn7yU4p5",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2397.223484456513,
			"y": 3402.1192917295866,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 207.23983764648438,
			"height": 25,
			"seed": 702222503,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726380,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "General Informtation",
			"rawText": "General Informtation",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "General Informtation",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 1212,
			"versionNonce": 2076218405,
			"isDeleted": false,
			"id": "-qrdiWSzcu7tOZwdvoxXq",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2364.5363235305585,
			"y": 3593.171244082914,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 1980.990341855123,
			"height": 766.6184230093213,
			"seed": 1973445575,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "J5Qtg6mypiCbRRVC6HtEy",
				"focus": 0.4622882497027057,
				"gap": 27.976419864549825
			},
			"endBinding": {
				"elementId": "EcIx1ETw8rGg27ymc-dHg",
				"focus": 0.15339544827486648,
				"gap": 14.033451903374043
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-1980.990341855123,
					766.6184230093213
				]
			]
		},
		{
			"type": "image",
			"version": 624,
			"versionNonce": 224259243,
			"isDeleted": false,
			"id": "0TNxTqTHE0lneHaxpUPFW",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3393.5285158187207,
			"y": 4320.760050047101,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 949.6667175292969,
			"height": 308.16999443003675,
			"seed": 1066291943,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "vIsbvCunYqyJfqY3MwmuG",
					"type": "arrow"
				}
			],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "b89d1e5a681c16226e822d099adccea9943c46ee",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 1310,
			"versionNonce": 1900332933,
			"isDeleted": false,
			"id": "vIsbvCunYqyJfqY3MwmuG",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3371.5808011480576,
			"y": 4479.467325937369,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 418.9272531184665,
			"height": 747.0832366943354,
			"seed": 1882283527,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619442,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "0TNxTqTHE0lneHaxpUPFW",
				"focus": -0.8897727417416323,
				"gap": 21.947714670663117
			},
			"endBinding": {
				"elementId": "J5Qtg6mypiCbRRVC6HtEy",
				"focus": -0.42923415625630396,
				"gap": 20.689320496739583
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-418.9272531184665,
					-747.0832366943354
				]
			]
		},
		{
			"type": "image",
			"version": 617,
			"versionNonce": 1206984523,
			"isDeleted": false,
			"id": "09kyefGAxKufFTDXFU9gW",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3386.0068284776053,
			"y": 3725.050684702018,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 950.6665649414065,
			"height": 285.4358668930774,
			"seed": 713426215,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "KlrXPdAsXI6As2defktiq",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "475dbd3cde8352927e38726f39e6ae2066b35beb",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 468,
			"versionNonce": 1118539493,
			"isDeleted": false,
			"id": "i4MXcUI12tlXq_hOH8ALR",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3370.812726508205,
			"y": 3214.995417368034,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 1017.1665649414058,
			"height": 303.5567160006504,
			"seed": 2113456199,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "sjTLyI7mSFGyp1-IriJDM",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "3896f173e8bc9132ae468b06792e641bfd627386",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 423,
			"versionNonce": 985988587,
			"isDeleted": false,
			"id": "-nx96k6rOmnO1JPqxqKEs",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3368.062543402736,
			"y": 2597.9955699559246,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 970.8399770201136,
			"height": 326.8333587646485,
			"seed": 1483089767,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "6nBzTKuf6FooP0sA11hOE",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "e9b42ef1853a44baaa8fdd0170cc54ca3772b8b1",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 1064,
			"versionNonce": 2122476101,
			"isDeleted": false,
			"id": "6nBzTKuf6FooP0sA11hOE",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3344.8126349554705,
			"y": 2759.7572586056167,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 387.8847819905013,
			"height": 671.554061451408,
			"seed": 941269639,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "-nx96k6rOmnO1JPqxqKEs",
				"focus": 0.8789557202021,
				"gap": 23.249908447265625
			},
			"endBinding": {
				"elementId": "J5Qtg6mypiCbRRVC6HtEy",
				"focus": 0.4390804801792529,
				"gap": 15.383448689269699
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-387.8847819905013,
					671.554061451408
				]
			]
		},
		{
			"type": "arrow",
			"version": 1024,
			"versionNonce": 1893992587,
			"isDeleted": false,
			"id": "sjTLyI7mSFGyp1-IriJDM",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3356.062726508205,
			"y": 3359.375828194695,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 320.833251953125,
			"height": 156.8213407133187,
			"seed": 500349351,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "i4MXcUI12tlXq_hOH8ALR",
				"focus": 0.6573908222128305,
				"gap": 14.75
			},
			"endBinding": {
				"elementId": "J5Qtg6mypiCbRRVC6HtEy",
				"focus": 0.351949066676244,
				"gap": 23.71673115997146
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-320.833251953125,
					156.8213407133187
				]
			]
		},
		{
			"type": "arrow",
			"version": 969,
			"versionNonce": 94820773,
			"isDeleted": false,
			"id": "KlrXPdAsXI6As2defktiq",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3363.562787543363,
			"y": 3870.8122846800643,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 334.7265719987431,
			"height": 249.1774001873829,
			"seed": 1052437703,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "09kyefGAxKufFTDXFU9gW",
				"focus": -0.752366279221785,
				"gap": 22.444040934242366
			},
			"endBinding": {
				"elementId": "J5Qtg6mypiCbRRVC6HtEy",
				"focus": -0.5534721408601606,
				"gap": 17.323472149511417
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-334.7265719987431,
					-249.1774001873829
				]
			]
		},
		{
			"type": "text",
			"version": 393,
			"versionNonce": 888869831,
			"isDeleted": false,
			"id": "bfe7QFT7",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3400.4066476898324,
			"y": 4272.256054383538,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 283.77972412109375,
			"height": 25,
			"seed": 1987034087,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726380,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "Coating defect : Insufficient",
			"rawText": "Coating defect : Insufficient",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Coating defect : Insufficient",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 442,
			"versionNonce": 2059808809,
			"isDeleted": false,
			"id": "c2czdslW",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3393.516989079807,
			"y": 3675.8671790580174,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 233.9197998046875,
			"height": 25,
			"seed": 2029889287,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726380,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "Coating defect : Bubble",
			"rawText": "Coating defect : Bubble",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Coating defect : Bubble",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 482,
			"versionNonce": 538116839,
			"isDeleted": false,
			"id": "lOsWTH5q",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3376.5024321950414,
			"y": 3168.922761740309,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 226.09982299804688,
			"height": 25,
			"seed": 1155608103,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726380,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "Coating defect : Crack",
			"rawText": "Coating defect : Crack",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Coating defect : Crack",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 514,
			"versionNonce": 1623212809,
			"isDeleted": false,
			"id": "hXWt6DGk",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3377.6346089121625,
			"y": 2558.9228736380956,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 266.1597900390625,
			"height": 25,
			"seed": 1426510151,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726381,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "NonCoating defect : Splash",
			"rawText": "NonCoating defect : Splash",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "NonCoating defect : Splash",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "image",
			"version": 471,
			"versionNonce": 1102917739,
			"isDeleted": false,
			"id": "1yMA0Ry9cE1M7XjtjnPfA",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2348.2027736138652,
			"y": 4703.484863739804,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 434.0484375,
			"height": 700.078125,
			"seed": 133852039,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "7nS6QUu05lbSTrSIZqEDN",
					"type": "arrow"
				},
				{
					"id": "OR_BFAO6-3IZL-KE-lhBf",
					"type": "arrow"
				},
				{
					"id": "MQ0Om5Ll8pbLfXKv4JO3-",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "4d96324e56095fdb25f0bbede3ea0b613ac33c65",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "text",
			"version": 639,
			"versionNonce": 1017122311,
			"isDeleted": false,
			"id": "vsgfsqbK",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2467.3354583760038,
			"y": 3282.3541934152554,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 477.17987060546875,
			"height": 45,
			"seed": 1355899559,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726381,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "COATING + NONCOATING",
			"rawText": "COATING + NONCOATING",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "COATING + NONCOATING",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "image",
			"version": 528,
			"versionNonce": 2133977867,
			"isDeleted": false,
			"id": "cNisEt5NcGXfhQWE3_cys",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1861.7763194270547,
			"y": 5587.316162039484,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 548,
			"height": 568,
			"seed": 1308390855,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "7nS6QUu05lbSTrSIZqEDN",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "c10d5bd095f84727de3818e9a5ed803366ea0080",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 445,
			"versionNonce": 897474341,
			"isDeleted": false,
			"id": "5D_qh1QvE32U88Gqa1pnB",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2741.193342132133,
			"y": 5582.649444510187,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 660,
			"height": 609,
			"seed": 379069671,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "OR_BFAO6-3IZL-KE-lhBf",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "75162803afca86f09621116dc4e258dc6e00d06c",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 1244,
			"versionNonce": 796283307,
			"isDeleted": false,
			"id": "7nS6QUu05lbSTrSIZqEDN",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2136.193342132133,
			"y": 5571.316314627375,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 397.91656494140625,
			"height": 152.08328247070312,
			"seed": 707850247,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "cNisEt5NcGXfhQWE3_cys",
				"focus": -0.7713483476055832,
				"gap": 15.999847412109375
			},
			"endBinding": {
				"elementId": "1yMA0Ry9cE1M7XjtjnPfA",
				"focus": -0.8171548506787852,
				"gap": 15.670043416867884
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					397.91656494140625,
					-152.08328247070312
				]
			]
		},
		{
			"type": "arrow",
			"version": 1216,
			"versionNonce": 1729180293,
			"isDeleted": false,
			"id": "OR_BFAO6-3IZL-KE-lhBf",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 3063.2764720149453,
			"y": 5569.232879568781,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 470.8331298828125,
			"height": 141.66656494140625,
			"seed": 1656968999,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "5D_qh1QvE32U88Gqa1pnB",
				"focus": 0.7814282923720636,
				"gap": 13.41656494140625
			},
			"endBinding": {
				"elementId": "1yMA0Ry9cE1M7XjtjnPfA",
				"focus": 0.8808562613838458,
				"gap": 24.00332588757101
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-470.8331298828125,
					-141.66656494140625
				]
			]
		},
		{
			"type": "arrow",
			"version": 983,
			"versionNonce": 867492939,
			"isDeleted": false,
			"id": "MQ0Om5Ll8pbLfXKv4JO3-",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2333.593412729385,
			"y": 4748.682893641495,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 1945.541868120612,
			"height": 295.8725654571681,
			"seed": 649845319,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "1yMA0Ry9cE1M7XjtjnPfA",
				"focus": 0.7038752691405481,
				"gap": 14.609360884480111
			},
			"endBinding": {
				"elementId": "EcIx1ETw8rGg27ymc-dHg",
				"focus": -0.06644807109788206,
				"gap": 18.53901483671143
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-1945.541868120612,
					-295.8725654571681
				]
			]
		},
		{
			"type": "image",
			"version": 280,
			"versionNonce": 968006117,
			"isDeleted": false,
			"id": "bDwwOtI6uWC0XNathcdgP",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2443.1618694916824,
			"y": 3983.423272661609,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 532,
			"height": 127,
			"seed": 1526135143,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "Dg1Ycu0GGmp5hmBBtd9_V",
					"type": "arrow"
				},
				{
					"id": "OiIb6JMUP_aDV6uxVfzzl",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "feb0e0e370f6f0ecbfb212196a2a14f8c6f23f16",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 571,
			"versionNonce": 1674060523,
			"isDeleted": false,
			"id": "Dg1Ycu0GGmp5hmBBtd9_V",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2421.0666895251643,
			"y": 4055.0185979499283,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 2026.1905343191966,
			"height": 349.99991280691984,
			"seed": 348206215,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "bDwwOtI6uWC0XNathcdgP",
				"focus": 0.3807244110263598,
				"gap": 22.095179966518117
			},
			"endBinding": {
				"elementId": "EcIx1ETw8rGg27ymc-dHg",
				"focus": 0.22714093202190688,
				"gap": 25.363625433906236
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-2026.1905343191966,
					349.99991280691984
				]
			]
		},
		{
			"type": "image",
			"version": 265,
			"versionNonce": 913315141,
			"isDeleted": false,
			"id": "QLg-tVM8mN4a3i29Dg_T5",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1966.8520550385538,
			"y": 4244.089939328276,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 1137,
			"height": 199,
			"seed": 198642599,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "OiIb6JMUP_aDV6uxVfzzl",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "d41e625d1af52bd2eb05a93fd3982e4bcc2a1bd9",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 505,
			"versionNonce": 943361419,
			"isDeleted": false,
			"id": "OiIb6JMUP_aDV6uxVfzzl",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2761.7688406373263,
			"y": 4238.065984338967,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 0,
			"height": 122.80697471217081,
			"seed": 1281687239,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "QLg-tVM8mN4a3i29Dg_T5",
				"focus": 0.3982705111675858,
				"gap": 6.023954989309459
			},
			"endBinding": {
				"elementId": "bDwwOtI6uWC0XNathcdgP",
				"focus": -0.19777056821670644,
				"gap": 4.8357369651866975
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					0,
					-122.80697471217081
				]
			]
		},
		{
			"type": "text",
			"version": 682,
			"versionNonce": 1739913705,
			"isDeleted": false,
			"id": "yuh1kGhR",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2615.587580611299,
			"y": 3913.320462589597,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 204.87596130371094,
			"height": 45,
			"seed": 688360935,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726381,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "THICKNESS",
			"rawText": "THICKNESS",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "THICKNESS",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "text",
			"version": 805,
			"versionNonce": 1436464423,
			"isDeleted": false,
			"id": "6qT36RYS",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 2545.359111852579,
			"y": 4639.177361306114,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 42.803985595703125,
			"height": 45,
			"seed": 1825257735,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726382,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "IC",
			"rawText": "IC",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "IC",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "image",
			"version": 326,
			"versionNonce": 335890437,
			"isDeleted": false,
			"id": "fzfpeMXThtJNdwIGnqy_I",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -566.6668357182143,
			"y": 3279.8531374698505,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 668,
			"height": 237,
			"seed": 1294762023,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "J5z6_N062tPNweiAdxnUh",
					"type": "arrow"
				},
				{
					"id": "D8Dk2rMDZCwxZQ0AoQFbJ",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "d81c58926674be9d2fde46f320a37e9c7d0a5ec5",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 364,
			"versionNonce": 265742027,
			"isDeleted": false,
			"id": "UlQjYD_MX1JHVVEBIEJ7m",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 386.3522818490592,
			"y": 3278.3531374698505,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 729,
			"height": 240,
			"seed": 471327559,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "6lcbYA0aVMYxCsH-R8O7d",
					"type": "arrow"
				},
				{
					"id": "OFsC6K3kEf0I7TEoQVpSf",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "aee86f6268e97a5d8266bac0f9cc2badbd0fb29b",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 302,
			"versionNonce": 713555813,
			"isDeleted": false,
			"id": "Xnisqn4yOvvUhZDsQPRNz",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1184.4815271832585,
			"y": 3264.8531374698505,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 667,
			"height": 267,
			"seed": 971629159,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "s_R7MPKleD31Xrl1zBDoB",
					"type": "arrow"
				},
				{
					"id": "VVzO4wcazVpq0n6IagEzW",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "42d6401c49f2ca80cbdf420805a3c0e7cd229298",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 313,
			"versionNonce": 562419051,
			"isDeleted": false,
			"id": "wjRPFUsuTHRkSK6HYHT4X",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -488.24918963643086,
			"y": 2682.7592939756073,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 719.9871210257397,
			"height": 254.151533647017,
			"seed": 1198456199,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "J5z6_N062tPNweiAdxnUh",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "b77b896361e32da5af61dd4599795148722690d4",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 307,
			"versionNonce": 41732805,
			"isDeleted": false,
			"id": "jOc6IsI3h_sryEQG2jlYU",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1311.8394354855764,
			"y": 2571.9107582644924,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 703.0442754442244,
			"height": 492.5151700106533,
			"seed": 400798887,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "s_R7MPKleD31Xrl1zBDoB",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "916dba59f86261d638efc6e5f392439369363492",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 320,
			"versionNonce": 144568331,
			"isDeleted": false,
			"id": "eILrY1khGEk3B_PQJrYrH",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1677.688830573451,
			"y": 2560.668343269819,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 1085,
			"height": 490,
			"seed": 1660192711,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "sEtgPVy6BatDd1wBCdgx6",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "e9226c55d4fb6744cd136668eaec647747b687e4",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 311,
			"versionNonce": 1283533349,
			"isDeleted": false,
			"id": "fy2hi-Sv7m88cjY2M91U_",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1418.8113768368721,
			"y": 3312.3531374698505,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 666,
			"height": 172,
			"seed": 918419175,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "sEtgPVy6BatDd1wBCdgx6",
					"type": "arrow"
				},
				{
					"id": "4J04GG3GDOI0WY8edJitg",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "00cfb228d3ef30068eb1b4c878cf91f26867342c",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 290,
			"versionNonce": 387561131,
			"isDeleted": false,
			"id": "ola10KVHo8kJqMrT7NBcU",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 336.17757232633176,
			"y": 2657.2669418684177,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 871.2222222222217,
			"height": 321.8028028028026,
			"seed": 701268487,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "6lcbYA0aVMYxCsH-R8O7d",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "dfc889e436978d1ac02bea99cd0658b5c157e168",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 510,
			"versionNonce": 1233296773,
			"isDeleted": false,
			"id": "sEtgPVy6BatDd1wBCdgx6",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1093.0346286420363,
			"y": 3071.385916295345,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 2.7777099609375,
			"height": 211.11104329427076,
			"seed": 1293723943,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "eILrY1khGEk3B_PQJrYrH",
				"focus": -0.08365113952249079,
				"gap": 20.71757302552578
			},
			"endBinding": {
				"elementId": "fy2hi-Sv7m88cjY2M91U_",
				"focus": -0.03449344539144363,
				"gap": 29.85617788023501
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-2.7777099609375,
					211.11104329427076
				]
			]
		},
		{
			"type": "arrow",
			"version": 537,
			"versionNonce": 2074323275,
			"isDeleted": false,
			"id": "J5z6_N062tPNweiAdxnUh",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -159.70129530870054,
			"y": 2954.719249628678,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 2.7777099609375,
			"height": 302.7777099609375,
			"seed": 1578822727,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "wjRPFUsuTHRkSK6HYHT4X",
				"focus": 0.08338835570408971,
				"gap": 17.808422006053434
			},
			"endBinding": {
				"elementId": "fzfpeMXThtJNdwIGnqy_I",
				"focus": 0.20560504148474626,
				"gap": 22.35617788023501
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-2.7777099609375,
					302.7777099609375
				]
			]
		},
		{
			"type": "arrow",
			"version": 532,
			"versionNonce": 1086235877,
			"isDeleted": false,
			"id": "6lcbYA0aVMYxCsH-R8O7d",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 773.6320380246316,
			"y": 3007.4969595896155,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 2.77791341145803,
			"height": 250,
			"seed": 950258535,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "ola10KVHo8kJqMrT7NBcU",
				"focus": 0.0005953361290492736,
				"gap": 28.427214918395066
			},
			"endBinding": {
				"elementId": "UlQjYD_MX1JHVVEBIEJ7m",
				"focus": 0.07413979905977632,
				"gap": 20.85617788023501
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					2.77791341145803,
					250
				]
			]
		},
		{
			"type": "arrow",
			"version": 522,
			"versionNonce": 731269099,
			"isDeleted": false,
			"id": "s_R7MPKleD31Xrl1zBDoB",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1648.6320380246316,
			"y": 3079.719249628678,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 5.555419921875,
			"height": 166.66666666666652,
			"seed": 151242375,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "jOc6IsI3h_sryEQG2jlYU",
				"focus": 0.06518125192876845,
				"gap": 15.293321353532065
			},
			"endBinding": {
				"elementId": "Xnisqn4yOvvUhZDsQPRNz",
				"focus": 0.4180246383974891,
				"gap": 18.467221174505994
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					5.555419921875,
					166.66666666666652
				]
			]
		},
		{
			"type": "arrow",
			"version": 515,
			"versionNonce": 901955653,
			"isDeleted": false,
			"id": "4J04GG3GDOI0WY8edJitg",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1095.8123386029738,
			"y": 3501.94143794248,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 1269.4443766276054,
			"height": 841.6665649414061,
			"seed": 1039621543,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "fy2hi-Sv7m88cjY2M91U_",
				"focus": 0.35927095364111544,
				"gap": 17.588300472629726
			},
			"endBinding": {
				"elementId": "EcIx1ETw8rGg27ymc-dHg",
				"focus": 0.45628054744267793,
				"gap": 25.253620720806794
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					1269.4443766276054,
					841.6665649414061
				]
			]
		},
		{
			"type": "arrow",
			"version": 505,
			"versionNonce": 217846411,
			"isDeleted": false,
			"id": "D8Dk2rMDZCwxZQ0AoQFbJ",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -268.0346286420345,
			"y": 3535.2747712758132,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 450,
			"height": 813.8888549804686,
			"seed": 984289479,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "fzfpeMXThtJNdwIGnqy_I",
				"focus": 0.2780142381620793,
				"gap": 18.421633805962756
			},
			"endBinding": {
				"elementId": "EcIx1ETw8rGg27ymc-dHg",
				"focus": 0.2552001002518254,
				"gap": 19.697997348411263
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					450,
					813.8888549804686
				]
			]
		},
		{
			"type": "arrow",
			"version": 551,
			"versionNonce": 1584164773,
			"isDeleted": false,
			"id": "OFsC6K3kEf0I7TEoQVpSf",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 734.7432847694245,
			"y": 3543.6081046091467,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 551.1902436755954,
			"height": 804.3650454566589,
			"seed": 1220239335,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "UlQjYD_MX1JHVVEBIEJ7m",
				"focus": -0.186750008133051,
				"gap": 25.25496713929624
			},
			"endBinding": {
				"elementId": "EcIx1ETw8rGg27ymc-dHg",
				"focus": -0.13006352439087154,
				"gap": 20.88847353888741
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-551.1902436755954,
					804.3650454566589
				]
			]
		},
		{
			"type": "arrow",
			"version": 545,
			"versionNonce": 1620019499,
			"isDeleted": false,
			"id": "VVzO4wcazVpq0n6IagEzW",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1520.8541246131745,
			"y": 3551.94143794248,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 1339.2855108351932,
			"height": 797.2221883138019,
			"seed": 1244973831,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Xnisqn4yOvvUhZDsQPRNz",
				"focus": -0.467739039167764,
				"gap": 20.088300472629726
			},
			"endBinding": {
				"elementId": "EcIx1ETw8rGg27ymc-dHg",
				"focus": -0.3595557539144227,
				"gap": 19.697997348410354
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-1339.2855108351932,
					797.2221883138019
				]
			]
		},
		{
			"type": "text",
			"version": 711,
			"versionNonce": 1622136009,
			"isDeleted": false,
			"id": "DMGEJCY2",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 131.32034165418645,
			"y": 3121.5526846872717,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 266.5079345703125,
			"height": 45,
			"seed": 731107879,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726382,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "OVERCOATING",
			"rawText": "OVERCOATING",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "OVERCOATING",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "text",
			"version": 834,
			"versionNonce": 578517063,
			"isDeleted": false,
			"id": "ZKBekOyX",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2819.6492712685367,
			"y": 2905.580105901581,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 462.99591064453125,
			"height": 45,
			"seed": 365229383,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726382,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "THICKNESS EX(TENSION)",
			"rawText": "THICKNESS EX(TENSION)",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "THICKNESS EX(TENSION)",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "image",
			"version": 312,
			"versionNonce": 565611109,
			"isDeleted": false,
			"id": "KRu7MrM3enmDSGTCHpcOw",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2931.7835853060433,
			"y": 3050.1358309992384,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 732,
			"height": 267,
			"seed": 1613018215,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "z_ZOB2uHSjIRP1aQDC8_P",
					"type": "arrow"
				},
				{
					"id": "P88whtN-r4Ozh6v_C--is",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "b4df204c0941095dde0093b1dd0a1bb2cb7345c8",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 325,
			"versionNonce": 1654225515,
			"isDeleted": false,
			"id": "jBx8BtCI-KvyLtzJo2sl0",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2934.7835853060433,
			"y": 3393.524994061924,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 738,
			"height": 288,
			"seed": 1115656071,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "iDP0eoGqzPs4wwu70mYLk",
					"type": "arrow"
				},
				{
					"id": "3jtpU_OUwPvr9_3FKp2f9",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "390366ff42810ea78c404a581ffb369a3ab648e1",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 330,
			"versionNonce": 1235350981,
			"isDeleted": false,
			"id": "ZzbBqt_0VJ6O-x-Ax6MbI",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2964.7835853060433,
			"y": 3787.76311836438,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 798,
			"height": 270,
			"seed": 1192363687,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "3E_PPyyglnKoR3WSW8y6c",
					"type": "arrow"
				},
				{
					"id": "_vw7emRXN8UAbM9S-1-7o",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "97820b633bef1f91f2088efca540b6901c944ab6",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 327,
			"versionNonce": 301266187,
			"isDeleted": false,
			"id": "oIHHMAwRaAl7j2x_tASqZ",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2933.7835853060433,
			"y": 4157.71526680188,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 736,
			"height": 192,
			"seed": 2089327047,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "A_eHAHD_NXKhf4q7w-1B6",
					"type": "arrow"
				},
				{
					"id": "xxI0L37132pHM9Mq1RvyF",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "5cb822b6f6e1db0beaf623f6392f1e2ffad090e0",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 368,
			"versionNonce": 1808332069,
			"isDeleted": false,
			"id": "yHKRibiumLneAMTVYHVYA",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -4234.979837851782,
			"y": 2733.788573452639,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 1093,
			"height": 387,
			"seed": 1744816359,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "z_ZOB2uHSjIRP1aQDC8_P",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "0904ce05555e407d2d7759025500d4af5be47bf7",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 334,
			"versionNonce": 1229462443,
			"isDeleted": false,
			"id": "JorSh9yGLlIDrtpgJ2wjO",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -4106.979837851782,
			"y": 3187.3407976363333,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 836.9999999999999,
			"height": 584,
			"seed": 923014151,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "iDP0eoGqzPs4wwu70mYLk",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "504b8b86bf6c7f7cf1b27a164d00f774c3bdb7ce",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 332,
			"versionNonce": 1670973573,
			"isDeleted": false,
			"id": "FKAY1PQpPsvsS1Q-R6VXB",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -4191.479837851782,
			"y": 3837.8930218200267,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 1006,
			"height": 375,
			"seed": 831650599,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "3E_PPyyglnKoR3WSW8y6c",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "182dcdaacf5e1b7ca6f0479378771c91c7dcfea9",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 321,
			"versionNonce": 79523403,
			"isDeleted": false,
			"id": "sD7nSxi602-wvF9EtC9il",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -4103.979837851782,
			"y": 4279.44524600372,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 831.0000000000001,
			"height": 346,
			"seed": 1813380679,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "A_eHAHD_NXKhf4q7w-1B6",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "20c28d0389488be998968f02c9cb10423d0c6b27",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 814,
			"versionNonce": 1120907237,
			"isDeleted": false,
			"id": "z_ZOB2uHSjIRP1aQDC8_P",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3130.8678795518435,
			"y": 2941.8897413107957,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 183.33333333333394,
			"height": 238.88885498046875,
			"seed": 963419495,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "yHKRibiumLneAMTVYHVYA",
				"focus": -0.7861962485058888,
				"gap": 11.111958299938124
			},
			"endBinding": {
				"elementId": "KRu7MrM3enmDSGTCHpcOw",
				"focus": -0.8102366546939136,
				"gap": 15.750960912466326
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					183.33333333333394,
					238.88885498046875
				]
			]
		},
		{
			"type": "arrow",
			"version": 816,
			"versionNonce": 1462725867,
			"isDeleted": false,
			"id": "iDP0eoGqzPs4wwu70mYLk",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3253.0903730414257,
			"y": 3500.2229729188684,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 297.22249348958394,
			"height": 52.777913411458485,
			"seed": 1558337671,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "JorSh9yGLlIDrtpgJ2wjO",
				"focus": -0.15404877487018076,
				"gap": 16.889464810356003
			},
			"endBinding": {
				"elementId": "jBx8BtCI-KvyLtzJo2sl0",
				"focus": -0.40445736414504263,
				"gap": 21.084294245798446
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					297.22249348958394,
					52.777913411458485
				]
			]
		},
		{
			"type": "arrow",
			"version": 808,
			"versionNonce": 125463365,
			"isDeleted": false,
			"id": "3E_PPyyglnKoR3WSW8y6c",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3164.2012128851757,
			"y": 4050.2229729188684,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 177.77750651041788,
			"height": 91.66666666666652,
			"seed": 2083762087,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "FKAY1PQpPsvsS1Q-R6VXB",
				"focus": 0.6605238352381745,
				"gap": 21.278624966606003
			},
			"endBinding": {
				"elementId": "ZzbBqt_0VJ6O-x-Ax6MbI",
				"focus": 0.5314977059166892,
				"gap": 21.640121068714507
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					177.77750651041788,
					-91.66666666666652
				]
			]
		},
		{
			"type": "arrow",
			"version": 826,
			"versionNonce": 338204555,
			"isDeleted": false,
			"id": "A_eHAHD_NXKhf4q7w-1B6",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3258.6457929633007,
			"y": 4455.778596291265,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 311.1112467447929,
			"height": 191.66666666666697,
			"seed": 1368203975,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "sD7nSxi602-wvF9EtC9il",
				"focus": 0.6250716344324712,
				"gap": 14.334044888481003
			},
			"endBinding": {
				"elementId": "oIHHMAwRaAl7j2x_tASqZ",
				"focus": 0.6965578323940684,
				"gap": 13.750960912464507
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					311.1112467447929,
					-191.66666666666697
				]
			]
		},
		{
			"type": "arrow",
			"version": 755,
			"versionNonce": 301382309,
			"isDeleted": false,
			"id": "P88whtN-r4Ozh6v_C--is",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2186.9312858337225,
			"y": 3184.234087582133,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 2133.469931509704,
			"height": 1167.4075469947502,
			"seed": 495661543,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "KRu7MrM3enmDSGTCHpcOw",
				"focus": -0.619302397104783,
				"gap": 12.852299472320738
			},
			"endBinding": {
				"elementId": "EcIx1ETw8rGg27ymc-dHg",
				"focus": -0.27141450208254647,
				"gap": 17.21998902780979
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					2133.469931509704,
					1167.4075469947502
				]
			]
		},
		{
			"type": "arrow",
			"version": 792,
			"versionNonce": 971206187,
			"isDeleted": false,
			"id": "3jtpU_OUwPvr9_3FKp2f9",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2186.705302633204,
			"y": 3542.678824118585,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 2128.098145540709,
			"height": 808.2775973687378,
			"seed": 1496997127,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "jBx8BtCI-KvyLtzJo2sl0",
				"focus": -0.48856019618546365,
				"gap": 10.078282672839123
			},
			"endBinding": {
				"elementId": "EcIx1ETw8rGg27ymc-dHg",
				"focus": -0.08040436584007864,
				"gap": 17.90520211737021
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					2128.098145540709,
					808.2775973687378
				]
			]
		},
		{
			"type": "arrow",
			"version": 778,
			"versionNonce": 1525816837,
			"isDeleted": false,
			"id": "_vw7emRXN8UAbM9S-1-7o",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2143.7710282891358,
			"y": 3933.361179726844,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 2087.9035556383387,
			"height": 421.6236834217493,
			"seed": 672605223,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "ZzbBqt_0VJ6O-x-Ax6MbI",
				"focus": -0.2587048116587781,
				"gap": 23.01255701690752
			},
			"endBinding": {
				"elementId": "EcIx1ETw8rGg27ymc-dHg",
				"focus": 0.10555483782433535,
				"gap": 13.876760456099873
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					1068.4592662038121,
					152.57629688901125
				],
				[
					2087.9035556383387,
					421.6236834217493
				]
			]
		},
		{
			"type": "arrow",
			"version": 752,
			"versionNonce": 457863371,
			"isDeleted": false,
			"id": "xxI0L37132pHM9Mq1RvyF",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2180.3726421922993,
			"y": 4254.329511200059,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 2119.743032264715,
			"height": 143.65081787109352,
			"seed": 168691527,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "oIHHMAwRaAl7j2x_tASqZ",
				"focus": 0.15495903617620418,
				"gap": 17.410943113743997
			},
			"endBinding": {
				"elementId": "EcIx1ETw8rGg27ymc-dHg",
				"focus": 0.4999772642780136,
				"gap": 17.142139699645668
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					980.8541046233458,
					-42.59854848859209
				],
				[
					2119.743032264715,
					101.05226938250144
				]
			]
		},
		{
			"type": "image",
			"version": 294,
			"versionNonce": 1768559973,
			"isDeleted": false,
			"id": "_DgNtbCCZCt0WT4s_qp3I",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3111.8545114366652,
			"y": 5071.724757434027,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 571.5711373223205,
			"height": 289.55582682291697,
			"seed": 1299940967,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "kuPR-1t8fPL-5krmkK-d5",
					"type": "arrow"
				},
				{
					"id": "fE19oYbgfLP9PrZJAavQp",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "f129603ccb702a7eab889284bdfb5ab9794301ea",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 293,
			"versionNonce": 1240997739,
			"isDeleted": false,
			"id": "Aqr0AkF8OI2igSKTSFOju",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2437.512761215313,
			"y": 5129.908801113685,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 647.2222222222247,
			"height": 173.18773946360218,
			"seed": 1093411207,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "-3k_psa_2IBi80kQVMzsN",
					"type": "arrow"
				},
				{
					"id": "hFlXN8gJTGWtKq6WAxge5",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "ba4f9196028c24bbc8289ca2b2392f14824502e5",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 307,
			"versionNonce": 145498309,
			"isDeleted": false,
			"id": "-eHRM_8Li0X5xeSUe5oIj",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1687.519926094058,
			"y": 5117.063012993195,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 662.6297200520839,
			"height": 198.8793157045818,
			"seed": 643586215,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "3vKaatF5ST9vju8hETjnq",
					"type": "arrow"
				},
				{
					"id": "HzcK-jOZ30e1ayy6gWFeB",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "d87c19820158696f3fd1a6d457b0ec4018efdc8a",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 303,
			"versionNonce": 1077344779,
			"isDeleted": false,
			"id": "QAjFw5ls9DS47q68yTBDg",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -922.1195931429438,
			"y": 5092.379062557022,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 560.7211839542825,
			"height": 248.2472165769282,
			"seed": 1656269767,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "xWLbHTWJWtaMVkrj0XwkN",
					"type": "arrow"
				},
				{
					"id": "GpvxkvTarPpf-cGuPq7HQ",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "a19825e6fb295b56b1bd15324fa4ff60aee08cf4",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 321,
			"versionNonce": 1045807141,
			"isDeleted": false,
			"id": "Auv4f_Nt0LybuG00qGqjf",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -258.627796289632,
			"y": 5112.458683585972,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 565.5925021701383,
			"height": 208.0879745190274,
			"seed": 1918737127,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "3QZJQzjv1IMVKrsvnDlIs",
					"type": "arrow"
				},
				{
					"id": "Cy-lDSEW3QcKKxTZgzKAK",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "e588f68bb8da6944042672e93135ea45ed163458",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 314,
			"versionNonce": 1258389675,
			"isDeleted": false,
			"id": "ovdYqr1kQXY2JbAstBf3q",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1064.1729238660691,
			"y": 5065.228477297099,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 664,
			"height": 302.5483870967742,
			"seed": 433563143,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "2Pv3NK_KL17ZAsdckTJch",
					"type": "arrow"
				},
				{
					"id": "tayEKHOA0faSQBcSHcylK",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "572f62bed669777905468c7d74fa7d85c752749a",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 341,
			"versionNonce": 5534597,
			"isDeleted": false,
			"id": "TUjv5Os47MZ5I80qepRpq",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 409.735318779537,
			"y": 5066.480999588742,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 551.6669921875003,
			"height": 300.0433425134894,
			"seed": 1478708519,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "ZyYLaYXjXWWfakgwsbiUu",
					"type": "arrow"
				},
				{
					"id": "fqDJ8bRvfbnVSHCKLmiVE",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "dcca27879461f85170ff5c1eab8d63846d8b1e06",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 238,
			"versionNonce": 872559435,
			"isDeleted": false,
			"id": "5jA_8FQtsa6OWvkWLsYHX",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -3172.999745019487,
			"y": 5466.9200378668575,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 655.0475027901775,
			"height": 109.29442417574862,
			"seed": 413995079,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "kuPR-1t8fPL-5krmkK-d5",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "c469d8ee22d2ee5c8405ca5164ac8276ea3db75f",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 508,
			"versionNonce": 719698661,
			"isDeleted": false,
			"id": "kuPR-1t8fPL-5krmkK-d5",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2846.547335002747,
			"y": 5455.928747756892,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 0.00017438615759601817,
			"height": 80.9524100167414,
			"seed": 626093927,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "5jA_8FQtsa6OWvkWLsYHX",
				"focus": -0.003270601266458102,
				"gap": 10.99129010996512
			},
			"endBinding": {
				"elementId": "_DgNtbCCZCt0WT4s_qp3I",
				"focus": 0.071658221727974,
				"gap": 13.695753483205863
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-0.00017438615759601817,
					-80.9524100167414
				]
			]
		},
		{
			"type": "arrow",
			"version": 516,
			"versionNonce": 731091435,
			"isDeleted": false,
			"id": "fE19oYbgfLP9PrZJAavQp",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2847.3007735807705,
			"y": 5047.544894840872,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 2983.33349609375,
			"height": 550,
			"seed": 1482484359,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "_DgNtbCCZCt0WT4s_qp3I",
				"focus": -0.875458147242869,
				"gap": 24.179862593155576
			},
			"endBinding": {
				"elementId": "EcIx1ETw8rGg27ymc-dHg",
				"focus": -0.7289672357342063,
				"gap": 17.68327123617928
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					2983.33349609375,
					-550
				]
			]
		},
		{
			"type": "image",
			"version": 252,
			"versionNonce": 915871301,
			"isDeleted": false,
			"id": "fs_Tidh9BfKV5ZrcpqYtj",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2464.700180123121,
			"y": 5417.368859735028,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 546.8461538461548,
			"height": 237.21951692695748,
			"seed": 244056487,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "-3k_psa_2IBi80kQVMzsN",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "9625d14fa0354f9fbe7b6dde82fdc13a0369c8d2",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 524,
			"versionNonce": 1925192843,
			"isDeleted": false,
			"id": "-3k_psa_2IBi80kQVMzsN",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2171.466805663903,
			"y": 5399.625207391278,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 15.051128233995769,
			"height": 87.82051282051316,
			"seed": 1493906631,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "fs_Tidh9BfKV5ZrcpqYtj",
				"focus": -0.012114499918533719,
				"gap": 17.743652343750455
			},
			"endBinding": {
				"elementId": "Aqr0AkF8OI2igSKTSFOju",
				"focus": 0.07735418515472471,
				"gap": 8.708153993477481
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					15.051128233995769,
					-87.82051282051316
				]
			]
		},
		{
			"type": "arrow",
			"version": 513,
			"versionNonce": 776340901,
			"isDeleted": false,
			"id": "hFlXN8gJTGWtKq6WAxge5",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -2126.62522821477,
			"y": 5109.086096832067,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 2272.222086588543,
			"height": 611.1111450195312,
			"seed": 1870070759,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Aqr0AkF8OI2igSKTSFOju",
				"focus": -0.6383656255410424,
				"gap": 20.82270428161837
			},
			"endBinding": {
				"elementId": "EcIx1ETw8rGg27ymc-dHg",
				"focus": -0.6207736634428802,
				"gap": 18.113328207842642
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					2272.222086588543,
					-611.1111450195312
				]
			]
		},
		{
			"type": "image",
			"version": 259,
			"versionNonce": 1161974571,
			"isDeleted": false,
			"id": "kuANgUw850fVABFjRs3bp",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1871.755219534215,
			"y": 5405.123246897171,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 813.3120812204813,
			"height": 259.0370822482641,
			"seed": 1641803527,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "3vKaatF5ST9vju8hETjnq",
					"type": "arrow"
				}
			],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "6d7b44461b641ec1387da8477e799e00d6424f7e",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 519,
			"versionNonce": 1696598277,
			"isDeleted": false,
			"id": "3vKaatF5ST9vju8hETjnq",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1412.5201269562322,
			"y": 5394.641720204463,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 38.10319048716519,
			"height": 61.11117892795119,
			"seed": 179729959,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619443,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "kuANgUw850fVABFjRs3bp",
				"focus": -0.07121668084532076,
				"gap": 10.48152669270803
			},
			"endBinding": {
				"elementId": "-eHRM_8Li0X5xeSUe5oIj",
				"focus": -0.1392155511714177,
				"gap": 17.588212578734783
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					38.10319048716519,
					-61.11117892795119
				]
			]
		},
		{
			"type": "arrow",
			"version": 517,
			"versionNonce": 59028939,
			"isDeleted": false,
			"id": "HzcK-jOZ30e1ayy6gWFeB",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1366.995372529008,
			"y": 5096.493560753508,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 1520.3704155815958,
			"height": 603.7037150065103,
			"seed": 1089673543,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "-eHRM_8Li0X5xeSUe5oIj",
				"focus": -0.5380747236132173,
				"gap": 20.569452239687052
			},
			"endBinding": {
				"elementId": "EcIx1ETw8rGg27ymc-dHg",
				"focus": -0.46984085584178004,
				"gap": 12.928222142305458
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					1520.3704155815958,
					-603.7037150065103
				]
			]
		},
		{
			"type": "image",
			"version": 251,
			"versionNonce": 1822597221,
			"isDeleted": false,
			"id": "qqylClGX7fbOrKWU6UdEv",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -1016.0460255435291,
			"y": 5440.505532275367,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 731.8237547892727,
			"height": 201.9090909090911,
			"seed": 1240356967,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "xWLbHTWJWtaMVkrj0XwkN",
					"type": "arrow"
				}
			],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "0a26c11a7ed0de37550ff2435ee87a94f3bc89ef",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 511,
			"versionNonce": 480363627,
			"isDeleted": false,
			"id": "xWLbHTWJWtaMVkrj0XwkN",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -628.6259152206558,
			"y": 5421.611444917413,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 38.32649935772406,
			"height": 61.61615545099448,
			"seed": 192197511,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "qqylClGX7fbOrKWU6UdEv",
				"focus": 0.2240606491078808,
				"gap": 18.894087357954504
			},
			"endBinding": {
				"elementId": "QAjFw5ls9DS47q68yTBDg",
				"focus": 0.32007544334181226,
				"gap": 19.36901033246795
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-38.32649935772406,
					-61.61615545099448
				]
			]
		},
		{
			"type": "image",
			"version": 292,
			"versionNonce": 1251691461,
			"isDeleted": false,
			"id": "Gst135XQ91AxvWBeNolQc",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -239.28709617943332,
			"y": 5418.138524313847,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 729.8333333333331,
			"height": 329.1877521085441,
			"seed": 2109909671,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "3QZJQzjv1IMVKrsvnDlIs",
					"type": "arrow"
				}
			],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "ade9f5336923b8644c1ec071e3d7d62aa68fa9e3",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 507,
			"versionNonce": 946077451,
			"isDeleted": false,
			"id": "GpvxkvTarPpf-cGuPq7HQ",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -617.5647044151165,
			"y": 5077.027328431685,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 766.6666666666661,
			"height": 583.333333333333,
			"seed": 1568980423,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "QAjFw5ls9DS47q68yTBDg",
				"focus": -0.3587782068880552,
				"gap": 15.351734125336407
			},
			"endBinding": {
				"elementId": "EcIx1ETw8rGg27ymc-dHg",
				"focus": -0.2763068580839858,
				"gap": 13.832371493659593
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					766.6666666666661,
					-583.333333333333
				]
			]
		},
		{
			"type": "arrow",
			"version": 486,
			"versionNonce": 1864862501,
			"isDeleted": false,
			"id": "3QZJQzjv1IMVKrsvnDlIs",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 85.21280209530141,
			"y": 5402.027328431685,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 77.77750651041606,
			"height": 72.2222900390625,
			"seed": 500657383,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Gst135XQ91AxvWBeNolQc",
				"focus": 0.28438976239802066,
				"gap": 16.111195882161155
			},
			"endBinding": {
				"elementId": "Auv4f_Nt0LybuG00qGqjf",
				"focus": 0.35140678467986414,
				"gap": 9.258380287622913
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-77.77750651041606,
					-72.2222900390625
				]
			]
		},
		{
			"type": "image",
			"version": 255,
			"versionNonce": 558228907,
			"isDeleted": false,
			"id": "l_iguYClfdsfwskwETlKP",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 543.9914581011608,
			"y": 5421.749567608118,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 534.8700000000001,
			"height": 315.0000000000001,
			"seed": 440875015,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "ZyYLaYXjXWWfakgwsbiUu",
					"type": "arrow"
				}
			],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "f0885f2cb273122a0105ac438048cdff4f6913e5",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 302,
			"versionNonce": 1790238341,
			"isDeleted": false,
			"id": "m30rptdJGjsEfww0vrujD",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 889.2962371538952,
			"y": 5859.833002666712,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 789,
			"height": 166.85264341957256,
			"seed": 147889959,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "2Pv3NK_KL17ZAsdckTJch",
					"type": "arrow"
				}
			],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "c507b88f496c7f0c65ce062b0656d9e166267dd5",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 499,
			"versionNonce": 1852962891,
			"isDeleted": false,
			"id": "ZyYLaYXjXWWfakgwsbiUu",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 800.6299672157429,
			"y": 5407.666400426044,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 145.83328247070312,
			"height": 31.25,
			"seed": 736109127,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "l_iguYClfdsfwskwETlKP",
				"focus": 0.7880060838840389,
				"gap": 14.083167182074249
			},
			"endBinding": {
				"elementId": "TUjv5Os47MZ5I80qepRpq",
				"focus": 0.7961969099658003,
				"gap": 9.892058323812762
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-145.83328247070312,
					-31.25
				]
			]
		},
		{
			"type": "arrow",
			"version": 595,
			"versionNonce": 1677772261,
			"isDeleted": false,
			"id": "2Pv3NK_KL17ZAsdckTJch",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1269.6212013724644,
			"y": 5832.666553013934,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 126.84235348976472,
			"height": 447.9167175292969,
			"seed": 1146241383,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "m30rptdJGjsEfww0vrujD",
				"focus": -0.10880245664978834,
				"gap": 27.16644965277783
			},
			"endBinding": {
				"elementId": "ovdYqr1kQXY2JbAstBf3q",
				"focus": -0.12788275633074564,
				"gap": 16.972971090764077
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					126.84235348976472,
					-447.9167175292969
				]
			]
		},
		{
			"type": "arrow",
			"version": 493,
			"versionNonce": 1841477355,
			"isDeleted": false,
			"id": "Cy-lDSEW3QcKKxTZgzKAK",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7.629875663008534,
			"y": 5084.7498354846375,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 150,
			"height": 583.3333740234375,
			"seed": 1131641991,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Auv4f_Nt0LybuG00qGqjf",
				"focus": -0.16287452531547797,
				"gap": 27.708848101334752
			},
			"endBinding": {
				"elementId": "EcIx1ETw8rGg27ymc-dHg",
				"focus": -0.06536826721819963,
				"gap": 21.55483785650722
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					150,
					-583.3333740234375
				]
			]
		},
		{
			"type": "arrow",
			"version": 500,
			"versionNonce": 1310245189,
			"isDeleted": false,
			"id": "fqDJ8bRvfbnVSHCKLmiVE",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 677.6298756630085,
			"y": 5051.4164614612,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 523.33349609375,
			"height": 546.6666259765625,
			"seed": 1023431591,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "TUjv5Os47MZ5I80qepRpq",
				"focus": 0.3578502881406872,
				"gap": 15.064538127541255
			},
			"endBinding": {
				"elementId": "EcIx1ETw8rGg27ymc-dHg",
				"focus": 0.3299804284874901,
				"gap": 24.88821187994472
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-523.33349609375,
					-546.6666259765625
				]
			]
		},
		{
			"type": "arrow",
			"version": 513,
			"versionNonce": 1304600971,
			"isDeleted": false,
			"id": "tayEKHOA0faSQBcSHcylK",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 1410.9633717567585,
			"y": 5044.7498354846375,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 1260,
			"height": 540,
			"seed": 421873351,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "ovdYqr1kQXY2JbAstBf3q",
				"focus": 0.6066618567110907,
				"gap": 20.47864181246132
			},
			"endBinding": {
				"elementId": "EcIx1ETw8rGg27ymc-dHg",
				"focus": 0.5941120607197639,
				"gap": 24.88821187994472
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-1260,
					-540
				]
			]
		},
		{
			"type": "text",
			"version": 790,
			"versionNonce": 1867273129,
			"isDeleted": false,
			"id": "t3MCXuA5",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -300.86419036346797,
			"y": 4976.101681119945,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 209.0519256591797,
			"height": 45,
			"seed": 108975591,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726383,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "UNDERFILL",
			"rawText": "UNDERFILL",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "UNDERFILL",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "rectangle",
			"version": 294,
			"versionNonce": 128727083,
			"isDeleted": false,
			"id": "lcc0pXxZ1YI9ohf1YPSYO",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -5063.677527050576,
			"y": 2126.5470978019275,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 9800,
			"height": 4799.9993896484375,
			"seed": 494742119,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false
		},
		{
			"type": "rectangle",
			"version": 252,
			"versionNonce": 1114829829,
			"isDeleted": false,
			"id": "t33caQjnIMmPIMRZOwIJF",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": -4827.8075370125525,
			"y": 4922.461294834743,
			"strokeColor": "#2f9e44",
			"backgroundColor": "transparent",
			"width": 1475,
			"height": 1458.3334350585938,
			"seed": 5425671,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false
		},
		{
			"type": "image",
			"version": 115,
			"versionNonce": 157940427,
			"isDeleted": false,
			"id": "4-O8WKw7ri8Uulm4t-AE0",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7222.681197506552,
			"y": 2768.142204240759,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 1109.921227478479,
			"height": 921.4671271127836,
			"seed": 1079229737,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "02370b1cec1018509df6b71607b61226b7cd9130",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 199,
			"versionNonce": 375401317,
			"isDeleted": false,
			"id": "Wc3sP-6TsBnU2YS5sl7QC",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6779.49427667858,
			"y": 3543.1546631631295,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 445.5025421626988,
			"height": 757.4071248372397,
			"seed": 943149383,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					275.6877983940967,
					-332.47339642237057
				],
				[
					445.5025421626988,
					-757.4071248372397
				]
			]
		},
		{
			"type": "image",
			"version": 168,
			"versionNonce": 529951083,
			"isDeleted": false,
			"id": "WIIbWkuC-D-Jh_Od-HUqW",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7297.800762293658,
			"y": 3765.538287527589,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 990,
			"height": 314,
			"seed": 2144464617,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "l0YH5ydy3KQoXYLeTYY4-",
					"type": "arrow"
				}
			],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "553111c6c118fd0bd573b70bb50dd7aa92d41357",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 484,
			"versionNonce": 443512517,
			"isDeleted": false,
			"id": "l0YH5ydy3KQoXYLeTYY4-",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7905.800762293658,
			"y": 2911.205066092042,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 883.3333740234375,
			"height": 848.3333435058594,
			"seed": 547130439,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "WIIbWkuC-D-Jh_Od-HUqW",
				"focus": -0.8934818900823953,
				"gap": 5.9998779296875
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					1.666748046875,
					166.66665649414062
				],
				[
					463.333251953125,
					288.3333435058594
				],
				[
					491.666748046875,
					818.3333435058594
				],
				[
					-151.666748046875,
					814.9999694824219
				],
				[
					-391.6666259765625,
					848.3333435058594
				]
			]
		},
		{
			"type": "image",
			"version": 271,
			"versionNonce": 528221195,
			"isDeleted": false,
			"id": "vwuxPJfnc9qE-8zBQV1ld",
			"fillStyle": "solid",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6113.505001983813,
			"y": 3799.63866330741,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 991.3535881420179,
			"height": 1038.9282739542807,
			"seed": 277626185,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "3e4c612bba5d3e49da99528062a9623fb76293aa",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "text",
			"version": 133,
			"versionNonce": 691951141,
			"isDeleted": false,
			"id": "TuczX1HX",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7149.552076002739,
			"y": 3686.5360889484264,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 123.13334655761719,
			"height": 423.9860948667967,
			"seed": 1957454089,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "ok5Z7-LY2NLJ9qGrqp4a_",
					"type": "arrow"
				}
			],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"fontSize": 368.6835607537363,
			"fontFamily": 2,
			"text": "{",
			"rawText": "{",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "{",
			"lineHeight": 1.15,
			"baseline": 339
		},
		{
			"type": "arrow",
			"version": 210,
			"versionNonce": 1360349867,
			"isDeleted": false,
			"id": "ok5Z7-LY2NLJ9qGrqp4a_",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7135.175926246462,
			"y": 3922.7746235521345,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 680,
			"height": 113.33331298828125,
			"seed": 853480457,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "TuczX1HX",
				"focus": -0.202920409876501,
				"gap": 14.3761497562773
			},
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-281.666748046875,
					-83.33331298828125
				],
				[
					-680,
					-113.33331298828125
				]
			]
		},
		{
			"type": "image",
			"version": 61,
			"versionNonce": 1840925061,
			"isDeleted": false,
			"id": "uKrILAmDfpAo94auy7T1f",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 11405.750248606166,
			"y": 1329.2023967173093,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 613.7331197070266,
			"height": 482.3529411764705,
			"seed": 1942185129,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "99HD9wvbur474sBsLpqdb",
					"type": "arrow"
				}
			],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "8da848eb342a91855d51ca815d190c836711706a",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 52,
			"versionNonce": 420265291,
			"isDeleted": false,
			"id": "99HD9wvbur474sBsLpqdb",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10973.597104875122,
			"y": 1570.5748978662064,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 425.49014820772027,
			"height": 1.9608082490808556,
			"seed": 77292585,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": null,
			"endBinding": {
				"elementId": "uKrILAmDfpAo94auy7T1f",
				"focus": -0.014846795295718621,
				"gap": 6.662995523323843
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					425.49014820772027,
					1.9608082490808556
				]
			]
		},
		{
			"type": "text",
			"version": 58,
			"versionNonce": 1495322853,
			"isDeleted": false,
			"id": "KRd87zSx",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 11109.675436154364,
			"y": 1587.2236891742778,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 162.140625,
			"height": 41.4,
			"seed": 235183719,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 2,
			"text": "Expanded",
			"rawText": "Expanded",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Expanded",
			"lineHeight": 1.15,
			"baseline": 33
		},
		{
			"type": "arrow",
			"version": 952,
			"versionNonce": 1656544235,
			"isDeleted": false,
			"id": "UBow9MxWuWjvmv6487H78",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10551.547162215003,
			"y": 1891.1500900167516,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 228.0164504278291,
			"height": 468.5713413783483,
			"seed": 701293063,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "exW4q81Cgs4vKwIPqKanB",
				"focus": 0.9203090172682014,
				"gap": 17.40088553253372
			},
			"endBinding": {
				"elementId": "OjkEhpOrvimlCPOIF0y9u",
				"focus": 0.9657193373267927,
				"gap": 25.312169329247354
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					189.761962890625,
					172.4601731073285
				],
				[
					228.0164504278291,
					468.5713413783483
				]
			]
		},
		{
			"type": "text",
			"version": 1256,
			"versionNonce": 957536103,
			"isDeleted": false,
			"id": "uTyeZVYt",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10392.22774967828,
			"y": 2182.367538908234,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 355.31982421875,
			"height": 35,
			"seed": 772868391,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726383,
			"link": null,
			"locked": false,
			"fontSize": 28,
			"fontFamily": 1,
			"text": "Called these in their body",
			"rawText": "Called these in their body",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Called these in their body",
			"lineHeight": 1.25,
			"baseline": 24
		},
		{
			"type": "image",
			"version": 603,
			"versionNonce": 1244669579,
			"isDeleted": false,
			"id": "MiIiN3nkPU4QOYWcO_HIT",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6856.973137150173,
			"y": 5308.529089974983,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 586.2169509555301,
			"height": 1056.493216055411,
			"seed": 974918087,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "IXecfWdO3iwS_8ogJrZDk",
					"type": "arrow"
				}
			],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "59757008b4d34cd7ed3a1f1baed5a5ddc33a369f",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 1197,
			"versionNonce": 1295227813,
			"isDeleted": false,
			"id": "IXecfWdO3iwS_8ogJrZDk",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6949.932634112449,
			"y": 6541.017379315557,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 204.5723889376875,
			"height": 450.3007299731677,
			"seed": 342114535,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Fp8KXkRaJbXSbQ-y0ecG8",
				"focus": 0.4235051878050441,
				"gap": 16.51497412689241
			},
			"endBinding": {
				"elementId": "MiIiN3nkPU4QOYWcO_HIT",
				"focus": 0.46217859400359435,
				"gap": 6.626153272873125
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-204.5723889376875,
					-131.9616291943472
				],
				[
					-99.5856502351487,
					-450.3007299731677
				]
			]
		},
		{
			"type": "image",
			"version": 261,
			"versionNonce": 102833451,
			"isDeleted": false,
			"id": "Fp8KXkRaJbXSbQ-y0ecG8",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6816.477416628568,
			"y": 6557.53235344245,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 619,
			"height": 630,
			"seed": 1343851527,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "IXecfWdO3iwS_8ogJrZDk",
					"type": "arrow"
				}
			],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "02599db85914c06fc5008454440cd927947300af",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "text",
			"version": 276,
			"versionNonce": 934672009,
			"isDeleted": false,
			"id": "rwZ2mnxZ",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6660.5872105108465,
			"y": 6416.786987861274,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 51.119964599609375,
			"height": 25,
			"seed": 2033742631,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726383,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "LIST",
			"rawText": "LIST",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "LIST",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 272,
			"versionNonce": 1068585607,
			"isDeleted": false,
			"id": "ZCsWt4MC",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6954.673617032768,
			"y": 5265.219638169721,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 413.69976806640625,
			"height": 25,
			"seed": 1747809863,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726384,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "Get data from TB_CCI_XXX_OVERVIEW",
			"rawText": "Get data from TB_CCI_XXX_OVERVIEW",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Get data from TB_CCI_XXX_OVERVIEW",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 528,
			"versionNonce": 1286094185,
			"isDeleted": false,
			"id": "q4VuAbd0",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6970.5651501246175,
			"y": 6519.941475379921,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 609.4794921875,
			"height": 25,
			"seed": 869602663,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726384,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "Get data from TB_CCI_XXX_<Corresponding Measure Item>",
			"rawText": "Get data from TB_CCI_XXX_<Corresponding Measure Item>",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Get data from TB_CCI_XXX_<Corresponding Measure Item>",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "image",
			"version": 216,
			"versionNonce": 399904363,
			"isDeleted": false,
			"id": "_tUEAzvEqsmgDXHybzu9B",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6901.89115351482,
			"y": 7336.541184657195,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 447,
			"height": 419,
			"seed": 1669058695,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "2cmQp5pwwdUt_-vZq7W9H",
					"type": "arrow"
				}
			],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "45e636a8707ed34f6cf2bf17c0b76a0195112116",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 465,
			"versionNonce": 52994501,
			"isDeleted": false,
			"id": "2cmQp5pwwdUt_-vZq7W9H",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 7361.317124651972,
			"y": 7356.041116840355,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 174.07389322916652,
			"height": 780.6878274584574,
			"seed": 1625171879,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "_tUEAzvEqsmgDXHybzu9B",
				"focus": 0.24418740083295226,
				"gap": 12.425971137152374
			},
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					174.07389322916652,
					-231.48152669270848
				],
				[
					57.671867249503975,
					-780.6878274584574
				]
			]
		},
		{
			"type": "image",
			"version": 269,
			"versionNonce": 444533003,
			"isDeleted": false,
			"id": "66v4iHigKalIuOPqWcGvZ",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8342.159125358588,
			"y": 6106.092067376752,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 696.8714634339105,
			"height": 624.8061621480873,
			"seed": 146040009,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "626a6128e4781d84a26dd8bab91d43123874eecd",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "text",
			"version": 187,
			"versionNonce": 1595107751,
			"isDeleted": false,
			"id": "aSOyQ6Py",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8480.931193717277,
			"y": 6072.190623842241,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 406.979736328125,
			"height": 25,
			"seed": 1212081065,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726384,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "Get data from KY_AOI.dbo.TB_AOIPCB",
			"rawText": "Get data from KY_AOI.dbo.TB_AOIPCB",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Get data from KY_AOI.dbo.TB_AOIPCB",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "image",
			"version": 280,
			"versionNonce": 1601154987,
			"isDeleted": false,
			"id": "MRCo5iCnoia7rG6RtRFhm",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9796.170086604925,
			"y": 5597.539375402961,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 643.0000000000001,
			"height": 316,
			"seed": 1388141353,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "94f601f9bb111eafe30aed90f5e69a65415e0cfe",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 877,
			"versionNonce": 1415885957,
			"isDeleted": false,
			"id": "WEax7xLePcjpwxGaNIi7R",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9981.05287259664,
			"y": 6115.159393827469,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 255.88258254119364,
			"height": 421.85860359556887,
			"seed": 647630345,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "22FXLsPDFUg6Czv62G9uf",
				"focus": 0.5695458326882255,
				"gap": 14.62219113251922
			},
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-255.88258254119364,
					-98.78676138511946
				],
				[
					-151.60814371717206,
					-421.85860359556887
				]
			]
		},
		{
			"type": "image",
			"version": 325,
			"versionNonce": 416449099,
			"isDeleted": false,
			"id": "22FXLsPDFUg6Czv62G9uf",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9853.865772640236,
			"y": 6129.781584959988,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 557,
			"height": 488,
			"seed": 102589673,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "WEax7xLePcjpwxGaNIi7R",
					"type": "arrow"
				}
			],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "a31d109a60f030cbcc60ddf2e1a11037709a604c",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "text",
			"version": 226,
			"versionNonce": 1567533129,
			"isDeleted": false,
			"id": "UbHAlWdS",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9654.283843184689,
			"y": 6017.07573837505,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 51.119964599609375,
			"height": 25,
			"seed": 215146441,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726384,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "LIST",
			"rawText": "LIST",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "LIST",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "image",
			"version": 229,
			"versionNonce": 1333919979,
			"isDeleted": false,
			"id": "YC5OiJTQgJwqA-o80Hm5q",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9836.82129888653,
			"y": 6762.217772234953,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 616.0537761961249,
			"height": 683.3333333333333,
			"seed": 1490419369,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "xAYDJqJCWt8-d6bJxzNpD",
					"type": "arrow"
				}
			],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "70258370012729a3c0a4befd76ab0dfaaa140b7b",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 454,
			"versionNonce": 1649088325,
			"isDeleted": false,
			"id": "xAYDJqJCWt8-d6bJxzNpD",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9940.199409549512,
			"y": 6751.766858507794,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 244.29973356071207,
			"height": 186.77354426170086,
			"seed": 953303433,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "YC5OiJTQgJwqA-o80Hm5q",
				"focus": 0.33900095876157105,
				"gap": 10.450913727159332
			},
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-244.29973356071207,
					-186.77354426170086
				],
				[
					-59.35290950826908,
					-167.70522587987557
				]
			]
		},
		{
			"type": "text",
			"version": 249,
			"versionNonce": 1952816327,
			"isDeleted": false,
			"id": "89pnR2Ww",
			"fillStyle": "solid",
			"strokeWidth": 2,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9619.423037505932,
			"y": 6550.198971372363,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 51.119964599609375,
			"height": 25,
			"seed": 436414569,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726384,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "LIST",
			"rawText": "LIST",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "LIST",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 82,
			"versionNonce": 602191657,
			"isDeleted": false,
			"id": "VzPaWlpP",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8391.439157360759,
			"y": 5813.337556014128,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 620.6975708007812,
			"height": 142.9166412353516,
			"seed": 976916681,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726385,
			"link": null,
			"locked": false,
			"fontSize": 114.33331298828126,
			"fontFamily": 1,
			"text": "PCBITEMS",
			"rawText": "PCBITEMS",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "PCBITEMS",
			"lineHeight": 1.25,
			"baseline": 101
		},
		{
			"type": "freedraw",
			"version": 95,
			"versionNonce": 1168179755,
			"isDeleted": false,
			"id": "24f2JHc7rRwXWlQbZ7sca",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 6661.022185518312,
			"y": 5318.059820621875,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 733.3333333333321,
			"height": 2461.1111450195312,
			"seed": 444126375,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					5.555419921875,
					0
				],
				[
					0,
					0
				],
				[
					-16.66666666666788,
					5.55552164713572
				],
				[
					-55.55582682291788,
					27.77781168619822
				],
				[
					-111.11124674479288,
					88.88885498046875
				],
				[
					-155.55582682291788,
					161.11114501953125
				],
				[
					-211.11124674479288,
					244.4444783528652
				],
				[
					-266.66666666666697,
					355.5555216471357
				],
				[
					-294.444580078125,
					449.9998982747402
				],
				[
					-305.55582682291697,
					516.6665649414062
				],
				[
					-305.55582682291697,
					599.9998982747402
				],
				[
					-300,
					683.3332316080732
				],
				[
					-288.88916015625,
					761.1111450195312
				],
				[
					-277.77791341145894,
					816.6665649414062
				],
				[
					-277.77791341145894,
					877.7778116861982
				],
				[
					-277.77791341145894,
					938.8888549804688
				],
				[
					-277.77791341145894,
					988.8888549804688
				],
				[
					-277.77791341145894,
					1049.9998982747402
				],
				[
					-294.444580078125,
					1099.9998982747402
				],
				[
					-316.66666666666697,
					1166.6665649414062
				],
				[
					-350,
					1211.1111450195312
				],
				[
					-372.22249348958394,
					1233.3332316080732
				],
				[
					-416.66666666666697,
					1261.1111450195312
				],
				[
					-444.444580078125,
					1261.1111450195312
				],
				[
					-494.444580078125,
					1272.2221883138027
				],
				[
					-550,
					1261.1111450195312
				],
				[
					-588.88916015625,
					1255.5555216471357
				],
				[
					-611.111246744792,
					1238.8888549804688
				],
				[
					-622.2224934895839,
					1227.7778116861982
				],
				[
					-633.3333333333339,
					1199.9998982747402
				],
				[
					-638.88916015625,
					1172.2221883138027
				],
				[
					-633.3333333333339,
					1127.7778116861982
				],
				[
					-605.555826822917,
					1094.4444783528652
				],
				[
					-594.444580078125,
					1083.3332316080732
				],
				[
					-583.3333333333339,
					1083.3332316080732
				],
				[
					-555.555826822917,
					1083.3332316080732
				],
				[
					-511.11124674479197,
					1083.3332316080732
				],
				[
					-450,
					1083.3332316080732
				],
				[
					-422.22249348958394,
					1099.9998982747402
				],
				[
					-394.444580078125,
					1116.6665649414062
				],
				[
					-372.22249348958394,
					1144.4444783528652
				],
				[
					-361.11124674479197,
					1177.7778116861982
				],
				[
					-350,
					1205.5555216471357
				],
				[
					-322.22249348958394,
					1255.5555216471357
				],
				[
					-311.11124674479197,
					1305.5555216471357
				],
				[
					-300,
					1355.5555216471357
				],
				[
					-294.444580078125,
					1416.6665649414062
				],
				[
					-277.77791341145894,
					1488.8888549804688
				],
				[
					-266.66666666666697,
					1544.4444783528652
				],
				[
					-266.66666666666697,
					1605.5555216471357
				],
				[
					-266.66666666666697,
					1655.5555216471357
				],
				[
					-266.66666666666697,
					1716.6665649414062
				],
				[
					-266.66666666666697,
					1783.3332316080732
				],
				[
					-266.66666666666697,
					1838.8888549804688
				],
				[
					-266.66666666666697,
					1905.5555216471357
				],
				[
					-266.66666666666697,
					1983.3332316080732
				],
				[
					-266.66666666666697,
					2033.3332316080732
				],
				[
					-266.66666666666697,
					2072.2219848632812
				],
				[
					-266.66666666666697,
					2105.555318196615
				],
				[
					-266.66666666666697,
					2138.888651529948
				],
				[
					-261.11124674479197,
					2172.2219848632812
				],
				[
					-250,
					2194.444478352865
				],
				[
					-238.88916015625,
					2227.777811686198
				],
				[
					-216.66666666666788,
					2255.555318196615
				],
				[
					-194.444580078125,
					2283.333231608073
				],
				[
					-166.66666666666788,
					2305.555318196615
				],
				[
					-144.444580078125,
					2333.333231608073
				],
				[
					-77.77791341145894,
					2377.777811686198
				],
				[
					-55.55582682291788,
					2388.888651529948
				],
				[
					-27.77791341145894,
					2405.555318196615
				],
				[
					0,
					2416.6665649414062
				],
				[
					33.33333333333212,
					2427.777811686198
				],
				[
					50,
					2438.888651529948
				],
				[
					61.11083984375,
					2444.444478352865
				],
				[
					77.77750651041606,
					2449.99989827474
				],
				[
					88.88875325520712,
					2461.1111450195312
				],
				[
					94.44417317708212,
					2461.1111450195312
				],
				[
					94.44417317708212,
					2461.1111450195312
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "freedraw",
			"version": 85,
			"versionNonce": 579324421,
			"isDeleted": false,
			"id": "-708U2QqTSsoYAJPLMjEN",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10561.02218551831,
			"y": 5595.837632308073,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 344.44417317708394,
			"height": 1799.999999999999,
			"seed": 459781063,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"points": [
				[
					0,
					0
				],
				[
					0,
					-5.555419921875
				],
				[
					5.55582682291606,
					0
				],
				[
					11.11083984375,
					33.33333333333303
				],
				[
					16.66666666666606,
					77.77791341145803
				],
				[
					27.77750651041606,
					133.33333333333303
				],
				[
					33.33333333333394,
					211.11124674479197
				],
				[
					38.88916015625,
					288.88895670572947
				],
				[
					38.88916015625,
					377.77791341145803
				],
				[
					38.88916015625,
					416.66666666666697
				],
				[
					38.88916015625,
					455.55562337239553
				],
				[
					38.88916015625,
					500
				],
				[
					44.44417317708394,
					550
				],
				[
					55.55582682291606,
					600
				],
				[
					72.22249348958394,
					638.8889567057286
				],
				[
					88.88916015625,
					677.777913411458
				],
				[
					122.22249348958394,
					722.2222900390625
				],
				[
					155.55582682291606,
					761.1112467447911
				],
				[
					183.33333333333394,
					799.9999999999991
				],
				[
					211.11083984375,
					833.333333333333
				],
				[
					233.33333333333394,
					861.111246744792
				],
				[
					255.55582682291606,
					866.666666666667
				],
				[
					277.77750651041606,
					866.666666666667
				],
				[
					294.44417317708394,
					866.666666666667
				],
				[
					316.66666666666606,
					849.9999999999991
				],
				[
					333.33333333333394,
					833.333333333333
				],
				[
					333.33333333333394,
					816.666666666667
				],
				[
					333.33333333333394,
					799.9999999999991
				],
				[
					327.77750651041606,
					777.777913411458
				],
				[
					305.55582682291606,
					755.5556233723955
				],
				[
					288.88916015625,
					744.444580078125
				],
				[
					266.66666666666606,
					733.333333333333
				],
				[
					244.44417317708394,
					727.777913411458
				],
				[
					233.33333333333394,
					727.777913411458
				],
				[
					205.55582682291606,
					727.777913411458
				],
				[
					188.88916015625,
					738.8889567057286
				],
				[
					172.22249348958394,
					755.5556233723955
				],
				[
					155.55582682291606,
					783.333333333333
				],
				[
					150,
					811.111246744792
				],
				[
					138.88916015625,
					849.9999999999991
				],
				[
					133.33333333333394,
					877.777913411458
				],
				[
					133.33333333333394,
					911.111246744792
				],
				[
					133.33333333333394,
					955.5556233723955
				],
				[
					133.33333333333394,
					999.9999999999991
				],
				[
					133.33333333333394,
					1072.2222900390616
				],
				[
					133.33333333333394,
					1111.111246744792
				],
				[
					133.33333333333394,
					1155.5556233723955
				],
				[
					133.33333333333394,
					1194.444580078124
				],
				[
					133.33333333333394,
					1233.333333333333
				],
				[
					127.77750651041606,
					1266.666666666667
				],
				[
					127.77750651041606,
					1311.111246744791
				],
				[
					116.66666666666606,
					1349.999999999999
				],
				[
					116.66666666666606,
					1383.333333333333
				],
				[
					116.66666666666606,
					1411.111246744791
				],
				[
					116.66666666666606,
					1449.999999999999
				],
				[
					116.66666666666606,
					1477.777913411458
				],
				[
					116.66666666666606,
					1527.777913411458
				],
				[
					116.66666666666606,
					1561.111246744791
				],
				[
					105.55582682291606,
					1594.444580078124
				],
				[
					100,
					1622.222086588541
				],
				[
					100,
					1644.444580078124
				],
				[
					100,
					1666.666666666666
				],
				[
					88.88916015625,
					1677.777913411458
				],
				[
					77.77750651041606,
					1688.888753255208
				],
				[
					66.66666666666606,
					1699.999999999999
				],
				[
					61.11083984375,
					1716.666666666666
				],
				[
					44.44417317708394,
					1744.444580078124
				],
				[
					27.77750651041606,
					1761.111246744791
				],
				[
					16.66666666666606,
					1772.222086588541
				],
				[
					11.11083984375,
					1772.222086588541
				],
				[
					0,
					1772.222086588541
				],
				[
					0,
					1783.333333333333
				],
				[
					-11.11083984375,
					1794.444580078124
				],
				[
					-11.11083984375,
					1794.444580078124
				]
			],
			"lastCommittedPoint": null,
			"simulatePressure": true,
			"pressures": []
		},
		{
			"type": "text",
			"version": 73,
			"versionNonce": 54815719,
			"isDeleted": false,
			"id": "jjiujZ5v",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 4988.800098929769,
			"y": 6413.893052229948,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 979.6199340820312,
			"height": 145.00000000000006,
			"seed": 1295274953,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726385,
			"link": null,
			"locked": false,
			"fontSize": 116.00000000000004,
			"fontFamily": 1,
			"text": "Data in left grid",
			"rawText": "Data in left grid",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Data in left grid",
			"lineHeight": 1.25,
			"baseline": 102
		},
		{
			"type": "text",
			"version": 109,
			"versionNonce": 309717513,
			"isDeleted": false,
			"id": "1RQBrxdZ",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10949.40583410307,
			"y": 6323.609938235652,
			"strokeColor": "#ff0000",
			"backgroundColor": "transparent",
			"width": 1019.9879150390625,
			"height": 145.00000000000006,
			"seed": 399987817,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726385,
			"link": null,
			"locked": false,
			"fontSize": 116.00000000000004,
			"fontFamily": 1,
			"text": "Data in right grid",
			"rawText": "Data in right grid",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Data in right grid",
			"lineHeight": 1.25,
			"baseline": 102
		},
		{
			"type": "arrow",
			"version": 39,
			"versionNonce": 1247851371,
			"isDeleted": false,
			"id": "dbGqU_Yb1JenhTTd--jeC",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9734.881928666871,
			"y": 2947.4987253992795,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 1.8519422743047471,
			"height": 248.148193359375,
			"seed": 582746345,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "OjkEhpOrvimlCPOIF0y9u",
				"focus": 0.03890936155399098,
				"gap": 17.465124674932213
			},
			"endBinding": null,
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					1.8519422743047471,
					248.148193359375
				]
			]
		},
		{
			"type": "text",
			"version": 60,
			"versionNonce": 1740396295,
			"isDeleted": false,
			"id": "V2X4t0yI",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9762.659570810967,
			"y": 3054.9062571375607,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 669.7437744140625,
			"height": 45,
			"seed": 1036659785,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726386,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "Component changed so this will happen",
			"rawText": "Component changed so this will happen",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Component changed so this will happen",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "text",
			"version": 228,
			"versionNonce": 1315769867,
			"isDeleted": false,
			"id": "szNXAlDW",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9386.010124264787,
			"y": 3309.43778198343,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 696.392578125,
			"height": 41.4,
			"seed": 1008944265,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 2,
			"text": "[Component Draw Info will also be updated]",
			"rawText": "[Component Draw Info will also be updated]",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "[Component Draw Info will also be updated]",
			"lineHeight": 1.15,
			"baseline": 33
		},
		{
			"type": "arrow",
			"version": 149,
			"versionNonce": 1698360357,
			"isDeleted": false,
			"id": "n-f9iWo9xH1a5i4kvoo_E",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10837.162446909651,
			"y": 2701.523590931979,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 808.3669700309692,
			"height": 7.608574328419309,
			"seed": 1842971047,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "OjkEhpOrvimlCPOIF0y9u",
				"focus": 0.12034260461681796,
				"gap": 17.250657189468257
			},
			"endBinding": {
				"elementId": "-die05TMCGw0NY2uY8ePr",
				"focus": -0.00964641980910686,
				"gap": 22.777955598579865
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					808.3669700309692,
					7.608574328419309
				]
			]
		},
		{
			"type": "text",
			"version": 132,
			"versionNonce": 1109246185,
			"isDeleted": false,
			"id": "XStCAzAH",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10858.876066288658,
			"y": 2635.0957540552695,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 760.7157592773438,
			"height": 45,
			"seed": 1923658951,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726386,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "Component changed so this will also happen",
			"rawText": "Component changed so this will also happen",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Component changed so this will also happen",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "image",
			"version": 64,
			"versionNonce": 1773412229,
			"isDeleted": false,
			"id": "Joon_-0Cp9xbUo2MSYBXf",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 9248.194362628017,
			"y": 3234.024763519625,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 976.9373590489527,
			"height": 56.32011382543851,
			"seed": 158845543,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "bf04029209fa74d56bda23add31154fce1ddb84d",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 54,
			"versionNonce": 1388529483,
			"isDeleted": false,
			"id": "-die05TMCGw0NY2uY8ePr",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 11668.3073725392,
			"y": 2359.3088184519406,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 724.7660185768262,
			"height": 700.078125,
			"seed": 505657287,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "n-f9iWo9xH1a5i4kvoo_E",
					"type": "arrow"
				},
				{
					"id": "h4UTr2lkn29J8VC1PRZbG",
					"type": "arrow"
				}
			],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "f17275bb126c3e9ecf4d0dd65f5eda4ee3a62c42",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 157,
			"versionNonce": 594536165,
			"isDeleted": false,
			"id": "zlOQWSsxOGS3SPGepYGUE",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 11490.963272144203,
			"y": 3338.3166575596506,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 895,
			"height": 300,
			"seed": 17910025,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "h4UTr2lkn29J8VC1PRZbG",
					"type": "arrow"
				}
			],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "5e76ba11cff7bc56d5f8b5f4e4994a65a063f560",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 77,
			"versionNonce": 2031232491,
			"isDeleted": false,
			"id": "h4UTr2lkn29J8VC1PRZbG",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 12064.169705054788,
			"y": 3072.9360030939333,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 1.8519422743047471,
			"height": 248.148193359375,
			"seed": 974416167,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "-die05TMCGw0NY2uY8ePr",
				"focus": -0.08429106153592117,
				"gap": 13.549059641992699
			},
			"endBinding": {
				"elementId": "zlOQWSsxOGS3SPGepYGUE",
				"focus": 0.2871173713049216,
				"gap": 17.23246110634227
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					1.8519422743047471,
					248.148193359375
				]
			]
		},
		{
			"type": "text",
			"version": 176,
			"versionNonce": 1212800551,
			"isDeleted": false,
			"id": "b51NsIRG",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 11889.932219851551,
			"y": 3171.0108137326934,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 149.68792724609375,
			"height": 45,
			"seed": 1713252231,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726387,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "Call this",
			"rawText": "Call this",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Call this",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "image",
			"version": 137,
			"versionNonce": 994848907,
			"isDeleted": false,
			"id": "SVGIaaONPsXfQXe8ngk7u",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10594.114445144573,
			"y": 4073.035498919501,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 1530.7116951778041,
			"height": 328.33312988281295,
			"seed": 723904809,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "7sqZS7x9uiXWudabE4rRK",
					"type": "arrow"
				},
				{
					"id": "x8xYKwfY3lRx1BvHZcQEW",
					"type": "arrow"
				}
			],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "300f320d015d53a25ba5e31e434e9f9b0680bb09",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "image",
			"version": 186,
			"versionNonce": 322253221,
			"isDeleted": false,
			"id": "5N-sRvyQ3XQyP3vb9eM-I",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 8750.676191972594,
			"y": 3476.926356009403,
			"strokeColor": "transparent",
			"backgroundColor": "transparent",
			"width": 1500.1039027779088,
			"height": 1520.5514157030104,
			"seed": 414049831,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "7sqZS7x9uiXWudabE4rRK",
					"type": "arrow"
				}
			],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"status": "pending",
			"fileId": "9429cb2605d1565b462e589b8978ed64e751e46e",
			"scale": [
				1,
				1
			]
		},
		{
			"type": "arrow",
			"version": 54,
			"versionNonce": 1261168427,
			"isDeleted": false,
			"id": "7sqZS7x9uiXWudabE4rRK",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10584.897287871509,
			"y": 4238.143162593203,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 319.4445510137666,
			"height": 1.8519422743056566,
			"seed": 1839657993,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "SVGIaaONPsXfQXe8ngk7u",
				"focus": -0.03221517186187907,
				"gap": 9.217157273064004
			},
			"endBinding": {
				"elementId": "5N-sRvyQ3XQyP3vb9eM-I",
				"focus": -0.00698937550368924,
				"gap": 14.67264210723988
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					-319.4445510137666,
					-1.8519422743056566
				]
			]
		},
		{
			"type": "text",
			"version": 235,
			"versionNonce": 543669193,
			"isDeleted": false,
			"id": "bscfRPX9",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 10358.105794361358,
			"y": 4180.614942003331,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 149.68792724609375,
			"height": 45,
			"seed": 970633769,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726387,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 1,
			"text": "Call this",
			"rawText": "Call this",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Call this",
			"lineHeight": 1.25,
			"baseline": 32
		},
		{
			"type": "arrow",
			"version": 872,
			"versionNonce": 1058785739,
			"isDeleted": false,
			"id": "wO2SynyNqEkPmZH7WsyuO",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 11429.657207397675,
			"y": -250.72308626995869,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 363.85788228834826,
			"height": 131.34916688419145,
			"seed": 699982153,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "BjQoVjeh",
				"focus": -0.7869626103124238,
				"gap": 9.248549780326357
			},
			"endBinding": {
				"elementId": "pPrpn90n",
				"focus": -0.7463683609572854,
				"gap": 5.763886343504055
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					363.85788228834826,
					131.34916688419145
				]
			]
		},
		{
			"type": "text",
			"version": 630,
			"versionNonce": 1205436743,
			"isDeleted": false,
			"id": "J5zKJFfp",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 11756.540140362235,
			"y": -203.45054626856688,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 174.83984375,
			"height": 25,
			"seed": 1015009321,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1715236726387,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "Current ViewModel",
			"rawText": "Current ViewModel",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Current ViewModel",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "rectangle",
			"version": 980,
			"versionNonce": 2083936363,
			"isDeleted": false,
			"id": "x-lEVBwwKHoWNdj2qBZJq",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 11744.488692420528,
			"y": -219.7967001147207,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 239.62475667112446,
			"height": 151.81913261859685,
			"seed": 2051400457,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 3
			},
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false
		},
		{
			"type": "text",
			"version": 799,
			"versionNonce": 1567578793,
			"isDeleted": false,
			"id": "pPrpn90n",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 11799.278976029527,
			"y": -130.87431329260187,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 180.17984008789062,
			"height": 25,
			"seed": 2139527657,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [
				{
					"id": "wO2SynyNqEkPmZH7WsyuO",
					"type": "arrow"
				},
				{
					"id": "2YXwWU64gj26Z4wMmtvVO",
					"type": "arrow"
				}
			],
			"updated": 1715236726387,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 1,
			"text": "UpdateItemResult",
			"rawText": "UpdateItemResult",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "UpdateItemResult",
			"lineHeight": 1.25,
			"baseline": 18
		},
		{
			"type": "text",
			"version": 498,
			"versionNonce": 1615205131,
			"isDeleted": false,
			"id": "2xDYvaXW",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0.3533880465047048,
			"x": 11473.357528811219,
			"y": -222.85753950252004,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 264.58984375,
			"height": 23,
			"seed": 1296921801,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619444,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Get SelectedInspItem as para",
			"rawText": "Get SelectedInspItem as para",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Get SelectedInspItem as para",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "arrow",
			"version": 1154,
			"versionNonce": 1025554213,
			"isDeleted": false,
			"id": "2YXwWU64gj26Z4wMmtvVO",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 11986.819255368497,
			"y": -114.3783836389489,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 139.09073108254233,
			"height": 1.13480307990298,
			"seed": 926282665,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619445,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "pPrpn90n",
				"focus": 0.23912077317869007,
				"gap": 7.3604392510787875
			},
			"endBinding": {
				"elementId": "Uw3BTWRPbPdYz9ZQ-n1Km",
				"focus": -0.03792462125302188,
				"gap": 17.71808401035807
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					139.09073108254233,
					1.13480307990298
				]
			]
		},
		{
			"type": "text",
			"version": 436,
			"versionNonce": 734362027,
			"isDeleted": false,
			"id": "qLzWV5Yt",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 12156.12349019109,
			"y": -124.13838134565697,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 244.599609375,
			"height": 23,
			"seed": 292263561,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619445,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Send data to other Controls",
			"rawText": "Send data to other Controls",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Send data to other Controls",
			"lineHeight": 1.15,
			"baseline": 18
		},
		{
			"type": "ellipse",
			"version": 495,
			"versionNonce": 2024311429,
			"isDeleted": false,
			"id": "Uw3BTWRPbPdYz9ZQ-n1Km",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 12143.623693641606,
			"y": -158.8606205220891,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 267.1297878689229,
			"height": 90.27781168619794,
			"seed": 514448745,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [
				{
					"id": "2YXwWU64gj26Z4wMmtvVO",
					"type": "arrow"
				},
				{
					"id": "x8xYKwfY3lRx1BvHZcQEW",
					"type": "arrow"
				}
			],
			"updated": 1712906619445,
			"link": null,
			"locked": false
		},
		{
			"type": "arrow",
			"version": 360,
			"versionNonce": 736037963,
			"isDeleted": false,
			"id": "x8xYKwfY3lRx1BvHZcQEW",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 12418.530721607143,
			"y": -121.43300071070394,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 588.833401150172,
			"height": 4361.201613300978,
			"seed": 546670857,
			"groupIds": [],
			"frameId": null,
			"roundness": {
				"type": 2
			},
			"boundElements": [],
			"updated": 1712906619445,
			"link": null,
			"locked": false,
			"startBinding": {
				"elementId": "Uw3BTWRPbPdYz9ZQ-n1Km",
				"focus": -1.0685500370106604,
				"gap": 9.01596798970425
			},
			"endBinding": {
				"elementId": "SVGIaaONPsXfQXe8ngk7u",
				"focus": 0.03959180091831419,
				"gap": 9.355695700409342
			},
			"lastCommittedPoint": null,
			"startArrowhead": null,
			"endArrowhead": "arrow",
			"points": [
				[
					0,
					0
				],
				[
					241.28828568892095,
					1009.0906871448863
				],
				[
					304.48451556581495,
					4358.090502189867
				],
				[
					-284.34888558435705,
					4361.201613300978
				]
			]
		},
		{
			"type": "text",
			"version": 111,
			"versionNonce": 1319039461,
			"isDeleted": false,
			"id": "cXKkKCeQ",
			"fillStyle": "solid",
			"strokeWidth": 4,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0,
			"x": 12502.09802867616,
			"y": 4512.511727749787,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 122.0625,
			"height": 41.4,
			"seed": 1887152871,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619445,
			"link": null,
			"locked": false,
			"fontSize": 36,
			"fontFamily": 2,
			"text": "Sample",
			"rawText": "Sample",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Sample",
			"lineHeight": 1.15,
			"baseline": 33
		},
		{
			"type": "text",
			"version": 552,
			"versionNonce": 611142379,
			"isDeleted": false,
			"id": "NGVIX0mJ",
			"fillStyle": "hachure",
			"strokeWidth": 1,
			"strokeStyle": "solid",
			"roughness": 1,
			"opacity": 100,
			"angle": 0.3533880465047048,
			"x": 11436.267559085916,
			"y": -175.8068243327334,
			"strokeColor": "#1e1e1e",
			"backgroundColor": "transparent",
			"width": 321.279296875,
			"height": 23,
			"seed": 1597591689,
			"groupIds": [],
			"frameId": null,
			"roundness": null,
			"boundElements": [],
			"updated": 1712906619445,
			"link": null,
			"locked": false,
			"fontSize": 20,
			"fontFamily": 2,
			"text": "Get SelectedInspItemResult as para",
			"rawText": "Get SelectedInspItemResult as para",
			"textAlign": "left",
			"verticalAlign": "top",
			"containerId": null,
			"originalText": "Get SelectedInspItemResult as para",
			"lineHeight": 1.15,
			"baseline": 18
		}
	],
	"appState": {
		"theme": "dark",
		"viewBackgroundColor": "#ffffff",
		"currentItemStrokeColor": "#1e1e1e",
		"currentItemBackgroundColor": "transparent",
		"currentItemFillStyle": "solid",
		"currentItemStrokeWidth": 4,
		"currentItemStrokeStyle": "solid",
		"currentItemRoughness": 1,
		"currentItemOpacity": 100,
		"currentItemFontFamily": 2,
		"currentItemFontSize": 36,
		"currentItemTextAlign": "left",
		"currentItemStartArrowhead": null,
		"currentItemEndArrowhead": "arrow",
		"scrollX": 2300.299681310727,
		"scrollY": 679.0884469031303,
		"zoom": {
			"value": 0.1
		},
		"currentItemRoundness": "round",
		"gridSize": null,
		"gridColor": {
			"Bold": "#C9C9C9FF",
			"Regular": "#EDEDEDFF"
		},
		"currentStrokeOptions": null,
		"previousGridSize": null,
		"frameRendering": {
			"enabled": true,
			"clip": true,
			"name": true,
			"outline": true
		}
	},
	"files": {}
}
```
%%