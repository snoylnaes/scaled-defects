# Defects

Each block below is one defect key. A key is a short name used in the `defect`
column of `zerocopy.tsv`, `classes.tsv`, and `compile.tsv`; every model whose
failure has the same cause carries the same key. A block states the attribution
(model defect, compiler defect, or generator defect) and the fix site, so the key
answers "what is wrong and where does it get fixed" once instead of once per model.
A key that starts with `ignore:` means the failure is accepted for now and the rest
of the key is the reason; it is not a fix site and it does not expire on its own.

# ZeroCopy

## ascii-no-filled-trait

fixed-length ASCII element has no recognized Filled trait (FileForText throws)

first error: FileForText: fixed-length ASCII element 'Protocol Version' (address: 'packet.packetheader.session.protocolversion', bytes: 3) has no recognized Filled trait (expected Zeros, NullCharacters, or Spaces). Fix the model/compiler to add the correct Filled trait for this field.

## utf16-translation

Ascii element carries a Translation=Utf16 trait the generator does not classify

first error: Specified argument was out of the range of valid values. (Parameter 'Unknown Type: [Field] Header')

## floating-point-field

IEEE-754 double (Translation=FloatingPoint) field the generator does not classify

first error: Specified argument was out of the range of valid values. (Parameter 'Unknown Type: [Field] Current Funding')

## big-endian-enum-overlay

multi-byte big-endian integer enum cannot be exposed by direct memory overlay

first error: Integer enum 'Deal Source' (address: 'serverpacket.serversoupbintcppacket.serverpayload.sequenceddatapacket.sequencedmessage.orderexecutedmessage.dealsource') is 2 bytes and big-endian. ZeroCopy cannot expose a multi-byte big-endian enum directly from a memory overlay. Use a supported little-endian representation or add an endian-aware wrapper type.

## sbe-block-length-ambiguous

SBE message-header block-length field could not be uniquely identified

first error: Model 'SmallX.OrderBookFeed.Sbe.v2.2': message-header block-length field could not be uniquely identified in composite 'Message Header' (packet.optiqmessage.messageheader). The original message targets do not state one shared Size dependency for the root block length.

## accessor-missing-size-trait

fixed-layout accessor field is missing required Size trait

first error: Model 'A2X.A2XEquities.UdpHeader.Amd.v1': fixed-layout accessor field is missing required Size trait. This is a model/compiler defect. Field='Payload' (Field, packet.message.payload, 0 bytes).

## cs0031-sbyte-overflow

generates clean, fails build: CS0031 sbyte-constant overflow in the Types emitter

first error: Types/HhiIndicator.cs(17,15): error CS0031: Constant value '128' cannot be converted to a 'sbyte'.
Bse.BseIndia.Eobi.Fbe.v1.4, the 14 non-v11+ Eurex.T7.Eobi.Fbe versions, the 7
B3.B3Derivatives.BinaryUmdf.Sbe versions, and B3.GapDetection.BinaryUmdf.v1 (23
models) moved out 2026-09-14: the declared-message-characteristic slice's
Boe3/Empty-target dispatch gate removal let them reach and pass generation.
The v11.0-v15.0 family (9 models) now also generates clean (the same gate
removal reaches them too) but fails build on a pre-existing, unrelated Types
emitter defect: an enum member value of 128 does not fit the signed-byte
backing type it is emitted as, in Types/HhiIndicator.cs and
Types/PrevPriceHhiIndicator.cs. Reconfirmed 2026-09-14 by
`sj verify-zerocopy-all` (output/verify-zerocopy-binary-qpbuq396/results.json):
status build-failure, generation ok, build failed, for all 9.
Bse.BseIndia.Eti.Fbe.v1.6.14 joined 2026-09-15 (run
output/verify-zerocopy-binary-41u4ufo2): same Types-emitter sbyte-overflow
defect, a different enum (Types/IncrementDecrementStatus.cs, member
NoValue = 0x80).

## cs0102-duplicate-size

generates clean, fails build: CS0102 duplicate 'Size' enum member (RESOLVED 2026-09-15)

first error (was): Types/EfficientMmtPublicationMode.cs(54,22): error CS0102: The type 'EfficientMmtPublicationMode' already contains a definition for 'Size'.
The collision was not the composite's own size field (the prior note's
attribution was wrong): the emitted enum type collided with a model-owned
enum value or bit child also named 'Size' on the same element. Commit
b912baff fixed the duplicate-member emitter defect. The 40
Euronext.Optiq.MarketDataGateway.Sbe.* models now pass end to end (dropped
below). The 40 Euronext.Optiq.OrderEntryGateway.Sbe.* models generate and
build past this defect but now fail build on a different, still-open
defect - see the CS1061 'LpRoleOptional' bucket below. Confirmed by
`sj verify-zerocopy-all`, output/verify-zerocopy-binary-5n0q7q7h/results.json:
0 models remain on CS0102.

## cs1061-enum-value-accessor

generates clean, fails build: CS1061 'LpRoleOptional' does not contain a definition for 'Value'

first error: Messages/MassCancelMessage.cs(64,57): error CS1061: 'LpRoleOptional' does not contain a definition for 'Value' and no accessible extension method 'Value' accepting a first argument of type 'LpRoleOptional' could be found (are you missing a using directive or an assembly reference?)
All 40 Euronext.Optiq.OrderEntryGateway.Sbe.* models moved here 2026-09-15
from the now-resolved CS0102 bucket above (commit b912baff fixed the
duplicate-'Size' emitter defect and let these reach a later build stage).
The new failure is an enum-typed field where Files/Group/Components/Accessor.cs
emits '.Value' against a plain C# enum type instead of the enum's backing
value. Status build-failure, generation ok, build failed, for all 40.
Confirmed by `sj verify-zerocopy-all`,
output/verify-zerocopy-binary-5n0q7q7h/results.json.

## multiple-value-facts

element declares more than one Value fact with different data; generator refuses by design

first error: Element 'Secondary Exec Id' (packet.simpleopenframe.payload.executionreporttrademessage.secondaryexecid) declares more than one 'FixTag' value with different data ('8' and '527'). The generator emits one 'FixTag' constant per element.

## bitfield-no-endian

NEW:bitfield-no-endian

first error: Bitfield container 'Exec Inst' (address: 'packet.data.unsequencedmessage.sbemessage.payload.newordersinglemessage.execinst') is 2 bytes but has no explicit Endian trait. Add exactly one Endian trait with value Big or Little upstream.

## multi-packet-header

NEW:multi-packet-header (RESOLVED 2026-09-13)

The PacketHeader.Find fix removed this first error. Six models now pass end to
end. The other 24 models moved to their actual first-error buckets above.

## enum-multibit-bitfield

enum-valued multi-bit bitfield children not implemented

first error: Bitfield child 'Party Role' (address: 'packet.message.payload.orderaddmessage.tableselect1.partyrole') carries enum values, but enum-valued multi-bit bitfield children are not implemented.

## default-branch-case

flat dispatch does not support a default Branch case (RESOLVED 2026-09-14)

The flat-dispatch-rejections slice (docs/zerocopy-4/progress-flat-dispatch-rejections.md)
removed the default-Branch-case rejection: Branch.cs now emits a DefaultCase
instead of throwing. All 9 models generate and build clean. Reconfirmed by
`sj verify-zerocopy-all`, output/verify-zerocopy-binary-9fusdb97/results.json
(before removal: unexpected-generation-success=9, exactly these 9 models;
after removal: 0 unexpected). Removed from the active ledger:
  Cboe.GapDetection.Pitch.v1
  Cboe.GapDetection.Pitch.v2
  Cboe.GapDetection.Pitch.v4
  Cboe.GapDetection.Pitch.v5
  Cboe.GapDetection.Pitch.v6
  Nasdaq.GapDetection.Itch.v11
  Nasdaq.GapDetection.Itch.v3
  Nasdaq.GapDetection.Itch.v7
  Nasdaq.GapDetection.Itch.v8

## leaf-no-translation-or-endian

leaf field with no Translation trait, or no Endian trait on a multi-byte Integer

first error: Specified argument was out of the range of valid values. (Parameter 'Unknown integer type: [Field] Strike Price Mantissa')

## nullable-no-value-entry

Nullable trait with no Nullable value entry

first error: Element 'Execution Mode' has the Nullable trait but no Nullable value entry — expected a Values entry with type=Nullable to carry the sentinel data.

## unresolved-element-argument

unresolved 'element' ArgumentOutOfRangeException

first error: Specified argument was out of the range of valid values. (Parameter 'element')

## unresolved-value-argument

unresolved 'value' ArgumentOutOfRangeException

first error: Specified argument was out of the range of valid values. (Parameter 'value')

## bitfield-nonprimitive-width

bitfield container spans a non-whole-primitive width

first error: Bitfield container 'Flags' (address: 'packet.messages.message.payload.trademessage.flags') spans 57 bits, which is not a whole-primitive width (expected 8/16/32/64). Non-byte-aligned or over-wide bitfield containers have no C# primitive backing and would truncate child masks; fix the model/compiler upstream or add a wide-backed bitfield template.

## nonstandard-decimal-width

NEW:nonstandard-decimal-width

first error: Element '[Field] Stop Px' is a non-standard-width (99-byte) decimal field. NonStandard does not apply Factor scaling; fix the model/compiler upstream.

## duplicate-struct-name

NEW:duplicate-struct-name

first error: Model 'Jpx.FseEquities.MarketByOrder.Flex.v1.1': two emitted structs collide on C# declaration name 'PacketHeader' — 'udppacket.packetheader' and 'tcppacket.packetheader'. Message-level qualification was insufficient.

## char-enum-one-char

NEW:char-enum-one-char

first error: Character enum 'Quote Condition' (packet.message.payload.consolidatedsinglesidedquotemessage.quotecondition) value 'Empty Quote' has data 'OxOO'. A one-byte ASCII enum value must contain exactly one character.

## single-branch-case

single unconditional Branch case dispatch not supported by ZeroCopy

first error: Cannot emit ZeroCopy library artifacts for 'Cboe.GapDetection.Pitch.v3': the dispatch element carries a single unconditional Branch case with no Dependency parameter, so there is no discriminator field to key on. ZeroCopy does not support single-unconditional-case dispatch.

## length-disambiguated-dispatch

NEW:length-disambiguated-dispatch

first error: Model 'Cboe.TitaniumConsolidated.OneEquitiesTcp.Pitch.v1.4.13': dispatch element 'packet.messages.message.payload' is classified as LengthDisambiguatedDispatch. ZeroCopy does not support shared wire codes.

## mixed-size-rule-dispatch-targets

NEW:mixed-size-rule-dispatch-targets

first error: Model 'Siac.Opra.Output.Obi.v6.3.Hft': some dispatch targets of 'packet.message.payload' carry a Size rule and some do not.

## remaining-payload-no-size-rule

NEW:remaining-payload-no-size-rule (RESOLVED 2026-09-14)

The declared-message-characteristic slice's dispatch gate removal let
Imperative.IntelligentCross.DepthOfBook.Aspen.v1.11 reach and pass generation
and build.

## cs0103

CS0103

first error: error CS0103: The name '...

## cs0155

CS0155

first error: error CS0155: The type caught or thrown must be derived f...

## cs0246

CS0246

first error: error CS0246: The type or namespace name '...

## cs0266-ulong-to-long

CS0266

first error: error CS0266: Cannot implicitly convert type 'ulong' to 'long'. An explicit conv...

## cs0542-value-member-name

CS0542

first error: error CS0542: 'Value': member na...

## cs0029

CS0029

first error: error CS0029: Cannot implicitly convert type 'string' to 'decimal'

## cs1061-hsvf-value

CS1061 (RESOLVED 2026-09-15)

first error (was): error CS1061: '<Type>' does not contain a definition for 'Value' and no accessible extension method 'Value' accepting a first argument of type '<Type>' could be found
`sj verify-zerocopy-all` cannot signal a fix for a skip-build entry (build
is never attempted), so each of the 6 models was rebuilt by hand from the
retained generated project in output/verify-zerocopy-binary-5n0q7q7h: Box.
Options.Sola.Multicast.Hsvf.v1.5/v1.8/v1.9 and Tmx.Mx.Sola.Multicast.Hsvf.
v1.11/v1.13/v1.14 all build with 0 errors. Dropped from the ledger; all 6
now pass end to end.

## enum-nonnumeric-ascii-value

generator defect: integer-enum member with single-character ASCII data emits as a bare unquoted C# identifier

first error: Types/NoUnspecifiedUnitReplay.cs(22,15): error CS0103: The name 'T' does not exist in the current context. ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Enum/RawValue.cs:9 (`RawValue.For`) returns `value.Data` verbatim for every enum member, with no validation. The source PDF (e.g. Cboe/Usa/BzxOptions/BinaryOrderEntry/Cboe.BzxOptions.BinaryOrderEntry.Boe.v2.10.Pdf.xml, NoUnspecifiedUnitReplay enum) genuinely documents this 1-byte field as mostly-numeric (False=0, True=1) with one or more ASCII-letter sentinel values (Test=T, and for the v2.3.7/CboeEquities variant also UserRequested=U, EndOfDay=E, Administrative=A) — the model correctly carries these as Enum values with data 'T'/'U'/'E'/'A' on an Integer-translated field, this is not a spec transcription error. `Scaled.CSharp.ZeroCopy.CSharp.Value.Literal.Integer` (ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Value/Literal.cs:56-119) already handles exactly this shape for non-enum Constant values: it Ascii-quotes a single-character non-numeric literal via `Is.Ascii.Character` before falling back to `Parse`, which throws a clean "integer data is not decimal or 0x hexadecimal" error otherwise. `RawValue.For` has no equivalent path for enum members — it neither quotes a lone ASCII-letter datum as `(byte)'T'` nor throws; it just interpolates the raw string, producing invalid C#. Fix site: ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Enum/RawValue.cs:9, reuse or mirror `Literal`'s ascii-character-vs-integer branch. Covers: Cboe.BzxEquities.BinaryOrderEntry.Boe.v2.3, Cboe.BzxOptions.BinaryOrderEntry.Boe.v2.10, Cboe.C1Options.BinaryOrderEntry.Boe.v2.10, Cboe.C2Options.BinaryOrderEntry.Boe.v2.10, Cboe.CboeEquities.BinaryOrderEntry.Boe.v2.3.7 (CS1525, same cause — 4 bad members instead of 1), Cboe.EdgxEquities.BinaryOrderEntry.Boe.v2.3, Cboe.EdgxOptions.BinaryOrderEntry.Boe.v2.10.

## value-name-collision-enclosing-type

model defect: a Constant value shares its Name with the Type/Field that declares it

first error: Model 'Nasdaq.Common.Xmp.Tcp.v1.0': element 'packet.xmppacket.payload.tipdatapacket.messagedelimiter' declares a Constant value 'Message Delimiter' with data '10' that resolves to the name 'MessageDelimiter', which is the name of the type that would declare it. Rename the value upstream. Generator gate is `ValueDeclarationNameCollisionException` (ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Value/ValueDeclarationNameCollisionException.cs), firing as designed. Root cause is upstream: OmiSpecifications/Nasdaq/Common/Headers/Xmp/Xmp.Tcp.v1.0.Source.xml declares both `<Type><Name>Message Delimiter</Name>...` (line 351) and `<Value><Name>Message Delimiter</Name><Value>10</Value>...</Value>` (line 371) with the identical Name, so the emitted constant name collides with its own enclosing type name. Propose renaming the `<Value>` block's `<Name>` (e.g. to "Line Feed") at line 372 in OmiSpecifications; not applied.

## source-field-integer-not-ascii

model defect: a field whose every declared value is a single ASCII letter is classified Translation=Integer instead of Ascii

first error: Element 'Source' (packet.packetheader.source) value 'Incremental' data 'I' is unsupported: integer data is not decimal or 0x hexadecimal. `Scaled.CSharp.ZeroCopy.CSharp.Value.Literal.Integer` (ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Value/Literal.cs) is firing as designed — it is entitled to reject non-numeric data on an Integer-translated value. Root cause is upstream: OmiSpecifications/SmallX/Common/Headers/OrderBookFeed.PacketHeader.Udp.Source.xml declares `<Type><Name>Source</Name>...<Trait><Category>Translation</Category><Value>Integer</Value></Trait>...` (around line 121) while every one of its three `<Value>` entries (lines 145-167: Incremental='I', Snapshot='S', Index Snapshot='X') is a single ASCII letter with no numeric member at all — unlike the mixed numeric/ASCII enum-nonnumeric-ascii-value case above, nothing here needs an integer form. Propose changing the Source Type's Translation trait from Integer to Ascii; not applied. Covers: SmallX.Common.Headers.Sbe.v1, SmallX.OrderBookFeed.Sbe.v2.2 (both compile this shared header file).

## nested-dispatch-key-not-direct-child

generator defect: ZeroCopy dispatch key resolution requires the discriminator to be a direct child of the dispatch container

first error: Model 'Nasdaq.Utp.Snapshot.Utp.v3.0': nested dispatch key 'serverpacket.servertcppayload.sequenceddatapacket.messageheader.messagecategory' is not a direct child of container 'serverpacket.servertcppayload.sequenceddatapacket'. Thrown from ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Framing/InnerDispatchKey.cs. A known ZeroCopy dispatch-shape limitation, in the same family as `single-branch-case` and `length-disambiguated-dispatch`: the discriminator here is one level deeper (inside 'messageheader') than the dispatching container, and the generator's dispatch-key lookup only looks at direct children. Not yet traced to a specific line-level fix; flagging as a real ZeroCopy dispatch-shape gap rather than a model defect, since the model's messagecategory field is a legitimate nested discriminator.

## zc-packet-fixture-missing-tree-prefix

not a defect: pcap fixture folder was not reorganized after the model gained a second transport tree

first error: Packet Nasdaq.NsmEquities.TotalView.Itch.v5.0.2026/AddOrderNoMpidAttributionMessage.pcap exited 1 (`Usage: ...Test <pcap-file> <tree>`, `tree: TcpInitiator | TcpAcceptor | Udp`). OmiSpecifications commits `c2a3420b`/`a6d33c60`/`f96ce483` ("Update Nasdaq edits" / "Update nasdaq specs" / "Updtae some protocols to public") changed `Nasdaq.NsmEquities.TotalView.Itch.v5.0.2026.Reference.xml` and its `Edits.xml`; the recompiled model now declares three root trees (Client Tcp Packet, Server Tcp Packet, Mold Udp 64 Packet — `Models/Binary/Nasdaq/Nasdaq.NsmEquities.TotalView.Itch.v5.0.2026.binary.model.json`, model_hash changed from `ddc8b80d...` to `4c7cebd9...`), a legitimate shape for a protocol with separate client-request, server-response, and multicast-replay transports. `ZeroCopy/Scaled.CSharp.ZeroCopy/Files/Manager/Program/Elements.cs:9-12` correctly switches the emitted Test program to `TreeSelectedProgram`, which requires a `<tree>` argument, once a model has more than one walk. `Packets/Nasdaq/Nasdaq.NsmEquities.TotalView.Itch.v5.0.2026/*.pcap` still sits directly under the model folder instead of one level deeper under a `<prefix>` subfolder (`TcpInitiator`/`TcpAcceptor`/`Udp`), the layout `~/code/sean-tools/scripts/verify_binary_folder.py` (`fixtures`, around line 88-90) requires to pass a tree argument. All 8 pcaps under this model fail identically with the harness usage message, not a decode error. Not a model or generator defect. Blocked on: sorting these fixtures into per-tree subfolders (or capturing fresh per-tree fixtures) — owner with access to the `Packets/` corpus.

## zc-timestamp-unit-microseconds-unsupported

generator defect: two independent type-selection functions disagree on a Unix-epoch, Unit=Microseconds field, so the emitted property type does not match the emitted field type

first error: error CS0029: Cannot implicitly convert type 'ulong' to 'System.DateTime'. The field carries traits `Translation=Integer`, `SemanticType=Timestamp`, `Unit=Microseconds`, `Epoch=Unix` (for example `Tmx.TsxAlpha.QuantumFeedLevel2.Xmt.v2.1`'s `packet.body.bodymessage.businessmessage.orderbookedmessage.prioritytimestamp`, `Models/Binary/Tmx/Tmx.TsxAlpha.QuantumFeedLevel2.Xmt.v2.1.binary.model.json`). `ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Field/DataType.cs` (`DataType.For`) types the message-class property `DateTime` for any element that satisfies `Is.Timestamp.Type(element) && Is.Epoch.Unix(element)`, with no check on `Unit`. `ZeroCopy/Scaled.CSharp.ZeroCopy/Files/Type/Elements.cs` (`IntegerFor`) selects the field's own type file by checking `Is.Unix.Nanosecond.Timestamp`, `Is.Unix.Millisecond.Timestamp`, and `Is.Unix.Second.Timestamp` in turn, with no `Is.Unix.Microsecond.Timestamp` case; a microsecond field falls through to the plain `Integer.Element.For` builder, which decodes to `ulong` (`Types/PriorityTimeStamp.cs` in the generated project). The shared predicate this needs already exists — `ScaledGenerators/Binary/Scaled.Binary.Model.Operations/Traits/Types/Timestamp/UnixMicroseconds.cs` — but ZeroCopy never calls it and has no microsecond decode/encode component or type-file builder (`Files/Type/Integer/Components/` holds `UnixSecondDecode.cs`, `LittleEndianUnixMillisecondDecode.cs`, `BigEndianUnixMillisecondDecode.cs`, `LittleEndianUnixNanosecondDecode.cs`, `BigEndianUnixNanosecondDecode.cs`, and matching `*TimestampDeclaration.cs` files, but nothing for microseconds). Not a model defect: the model's traits are unremarkable and match the millisecond/nanosecond shapes the generator already supports. Fix site: `ZeroCopy/Scaled.CSharp.ZeroCopy/Files/Type/Elements.cs` (`IntegerFor`) needs an `Is.Unix.Microsecond.Timestamp` branch and a matching `Integer.LittleEndianUnixMicrosecondTimestamp` / `BigEndianUnixMicrosecondTimestamp` type-file builder with decode/encode components, mirroring the millisecond pair. Covers Tmx.TsxAlpha.QuantumFeedLevel2.Xmt.v2.1, Tmx.TsxAlpha.QuantumFeedLevel2.Xmt.v2.2. These two rows previously carried `unresolved-element-argument` (generation-stage failure); that defect was fixed upstream and generation now reaches build, exposing this masked second defect.

# Classes

## §15

UNSUPPORTED - multi-transport (multi-tree) models


## §16

nullable/default sentinel unsupported


## §18

one-byte length field


## §25

untriaged Classes-only InvalidOperationException (Siac.Opra.Recipient.Obi.v4.0)


## §27

Block Header loses its Header characteristic to a duplicate-identifier struct/type


## §28

B3.B3Derivatives.BinaryEntryPoint.Sbe.v8.4 CS0103


## §29

MessageTypeConstant '0x..' was expected to be a hex literal


## §30

IntegerType.For: wire width is not supported (2026-09-09: both members)

(Cboe.BxeEquities/CxeEquities.AuctionFeed.AsciiPitch.v1.4) now fail generation with
'Field 'Price' is a decimal field with unsupported width 19', the same message
template as §41 (decimal field unsupported width). Merged into §41.

## §33

CS0103 the name 'R' does not exist in the current context (2026-09-09:)

both members (Miax.PearlEquities.DepthOfMarket.Mach.v1.3.d,
Miax.PearlEquities.TopOfMarket.Mach.v1.1.c) now pass end to end; see the dropped list.

## §35

filed 2026-09-01, never denylisted


## §36

MemberFieldHex - fixed 2026-09-02


## §37

OverflowException: value too large or too small


## §39

MessageTypeConstant '<XX>' is not a valid literal


## §42

ArgumentNullException: Value cannot be null


## §47

dispatch element with an unsupported Branch shape


## §2B

String field has no recognized Filled trait (2026-09-12: both)

members (Omi.Sbe.Example.Sbe.v1, Omi.Sbe.Example.Sbe.v2) now pass end to end
(verify-classes-all unexpected-generation-success after the Classes 4 conversion);
confirmed with sj check-classes and sj diff-generated classes vs 3ec23c45 (only the
added Dispatch.cs 'using System.Buffers.Binary;' line differs). Ledger was stale.

## §61

multi-byte integer field has no Endian trait (This section was)

mislabeled §59 in the live block (a pre-existing collision with the unrelated,
still-active §59 FloatingPoint section below; renumbered here for clarity, not
re-triaged). 2026-09-12: both members (Cboe.C1Options.MarketDataFeed.Csm.v1.4.2,
Cboe.C1Options.OpeningAuction.Csm.v1.0) now pass end to end (verify-classes-all
unexpected-generation-success after the Classes 4 conversion); confirmed with sj
check-classes and sj diff-generated classes vs 3ec23c45 (byte-identical). Ledger
was stale.

## §54

group field offset does not match the enclosing message layout

2026-09-09: all 9 former members now fail earlier in generation with a different
error. The 7 B3.B3Derivatives.BinaryEntryPoint.Sbe.v7.0/v7.1/v8.0/v8.1/v8.2/v8.3/v8.4
models now raise 'Count field
...crosssidesgroups.groupsizeencoding.numingroup' is outside the parsed property
path...' (the §34 message template); moved to §34. The 2 Iex.IexOptions.MarketData.
Sbe.v1.03 / Iex.IexOptions.Session.Sbe.v1.0 models now raise 'Branch target ... is
neither a generated message nor a branch container with a direct dispatch child'
(the §53 message template); moved to §53.

## §20

duplicate-address sibling elements rejected by generator invariant

2026-09-13: all 14 members (Cboe.ByxEquities/BzxEquities/BzxOptions/C1Options/
C2Options/EdgaEquities/EdgxEquities/EdgxOptions.MulticastDepthOfBook.
Pitch/Spin.v2.41.66) now fail generation earlier ('Unsupported length field byte
width: 1. Expected 2 or 4.', the §38 template) against the freshly compiled
1083-model corpus; moved to §38.

## §26

CS0102 duplicate FutureLegList/ParseStarted/FailedPrefixField emission

2026-09-13: both members (Cboe.CfeFutures.MulticastDepthOfBook.Pitch.v1.1.12,
v1.1.6) now fail generation earlier ('Unsupported length field byte width: 1.
Expected 2 or 4.', the §38 template) against the freshly compiled 1083-model
corpus; moved to §38.

## §34

count field is outside the parsed property path

2026-09-13: the sole member (Cboe.C1Options.MarketLevel2.Csm.v1.0.4) now fails
generation earlier ('Branch target ... is neither a generated message nor a
branch container with a direct dispatch child', the §53 template) against the
freshly compiled 1083-model corpus; moved to §53.

## §48

CS0108 AdapFlagsField.Clear hides the inherited member

2026-09-13: all 4 members (Cboe.ByxEquities/BzxEquities/EdgaEquities/
EdgxEquities.SummaryDepth.Pitch.v1.0.4) now fail generation earlier ('Unsupported
length field byte width: 1. Expected 2 or 4.', the §38 template) against the
freshly compiled 1083-model corpus; moved to §38.

## §51

FormatException: input string was not in a correct format

2026-09-13: the sole member (Cboe.DxeDerivatives.MulticastDepthOfBook.Pitch.
v1.11) now fails generation earlier ('Unsupported length field byte width: 1.
Expected 2 or 4.', the §38 template) against the freshly compiled 1083-model
corpus; moved to §38.

## §60

leaf field with no Translation trait and no Endian trait; Field.Kind has nothing to dispatch on

2026-09-13: all 7 members now pass end to end (verify-classes-all
unexpected-generation-success against the freshly compiled 1083-model corpus).
Ledger was stale.

## §63

group field does not fit before the message dispatch field in a size-prefixed frame

2026-09-13: all 7 B3.B3Derivatives.BinaryEntryPoint.Sbe.v7.0/v7.1/v8.0/v8.1/
v8.2/v8.3/v8.4 members now pass end to end (verify-classes-all
unexpected-generation-success against the freshly compiled 1083-model corpus).
Ledger was stale.

## §52

String field has no recognized Justified trait

first error: String field 'ProtocolVersion' has no recognized Justified trait. Fix the model/compiler upstream.

## §43

Ascii element carries a Translation=Utf16 trait the generator does not classify

first error: Field.Kind: cannot classify element. The field is UTF-16-encoded text (Translation=Utf16); Field.Kind has no Utf16 branch. This is the

## §59

IEEE-754 double (Translation=FloatingPoint) field the generator does not classify

first error: Field.Kind: cannot classify element. Address='packet.message.payload.newordermessage.orderid', Name='Order Id', ByteWidth=8,

## §32

SentinelLiteral.For: archetype is not supported

first error: SentinelLiteral.For: archetype 'Bitfield' is not supported for field 'MatchEventIndicatorMatchEventIndicatorOptional'.

## §24

positive decimal exponent unsupported

first error: Element 'Trading Value' has a positive decimal exponent (3). Positive exponents require multiply-on-parse semantics; update the generator templates before addin...

## §38

size dependency field width unsupported

first error: Unsupported length field byte width: 1. Expected 2 or 4.
2026-09-13: reworded again (was 'Unsupported wire primitive byte width: 1. Expected
  2, 4, or 8.'); same underlying guard, wording has flipped between these two forms
  across passes. Also broadened sharply: grew from 9 to 126 members, absorbing all of
  former §20 (14), §26 (2), §48 (4), and §51 (1), plus 98 models that generated
  cleanly before this pass. See the top-of-file 2026-09-13 note.

## §41

decimal field with unsupported width

first error: Field 'PricePrice10' is a decimal field with unsupported width 10.

## §53

Branch target is neither a generated message nor a branch container

first error: Model 'B3.B3Derivatives.BinaryUmdf.Sbe.v1.6': Branch target 'Sequence Reset Message' (packet.message.payload.sequenceresetmessage) is neither a generated messag...

Same root cause now also surfaces at build instead of generation for 93 mostly
Nasdaq/Tadawul/Odx/NsxAustralia session-control branch targets (ClientHeartbeat,
ClientHeartbeatPacket, AdminHeartbeat, SubscriberHeartbeat, SequenceResetMessage,
and related empty session messages): generation stopped throwing the §53
exception for these, so it emits Framing/Dispatch.cs with a `case ...: return
X.Parse(payload);` call to a branch-target class X that was never generated,
producing error CS0103 at build. The classification gap that produces §53 was
never fixed, only unmasked at a later stage; every classes.tsv row here carries
prev_stage=generation, prev_defect=§53.

## §55

counted element has a non-integer count field

first error: Count field 'packet.messagebody.complexorderinstrumentkeysmessage.numberoflegs' must use a standard integer width.

## §56

message count dependency field is not inside the packet header

first error: Model 'A2X.A2XEquities.Rtmdf.Amd.v1.3.2': message count dependency field 'Message Count' (packet.messagecount) is not inside packet header 'Message Header' (packet.message.messageheader).
2026-09-13: all 6 former Siac Cqs/Cts members now pass end to end
  (unexpected-generation-success); the 1 remaining former member, Siac.Opra.
  Output.Obi.v4.0, now fails earlier ('Branch rule on element ... has 15
  Dependency parameters; at most one is valid', the §31 template) and moved to §31.
  2 new A2X models take its place on this same message-count-header shape.

## §45

enum-valued multi-bit bitfield children not implemented

first error: Bitfield child 'Party Role' (address: 'packet.message.payload.ioiaddmessage.tableselect1.partyrole') carries enum values, but enum-valued multi-bit bitfield chi...

## §44

message has no matching branch case in its dispatch element

first error: Message 'Delta Update Message' (packet.payload.deltaupdatemessages.deltaupdatemessage) has no matching branch case in dispatch element 'packet.payload'. Every d...

## §31

branch rule carries an unexpected number of Dependency parameters

first error: Branch rule on element 'Equity And Index Last Sale Message Payload' (packet.message.payload.equityandindexlastsalecategory.equityandindexlastsalemessagepayload)...

## §46

composite-key dispatch target has no body

first error: Model 'Siac.Cqs.Output.Cta.v2.10.Hft': composite-key dispatch target 'packet.message.messagepayload.startofdaymessage' has no body.

## §50

Bitfield.BackingType: unsupported byte width

first error: Field 'Flags' is a bitfield with unsupported width 7.

## §62

field has no generated shared field class for its wire digest

first error: Field 'pillarstreammessage.seqmsg.sequencedmessage.newordermessage.optionalorderaddon.

## §40

unresolved ArgumentOutOfRangeException

first error: Specified argument was out of the range of valid values. (Parameter 'element')

## §64

frame-header size dependency field is not 2 or 4 bytes wide

first error: Model 'Aquis.AquisEquities.Replay.Amd.v4.0': size dependency field 'Msg Length' (1 bytes) must be 2 or 4 bytes wide.
2026-09-13: the former sole member, SmallX.OrderBookFeed.Sbe.v2.2, now passes end to
  end (unexpected-generation-success); 2 new Aquis models take its place on this same
  frame-header shape.

## §65

dispatch collision targets share one payload length

first error: Model 'Cboe.TitaniumConsolidated.OneEquitiesTcp.Pitch.v1.4.13': dispatch collision targets share one payload length.

## §66

branch element has Count, Payload(Buffer=Rest), and Branch rules but no Size rule

first error: Model 'Imperative.IntelligentCross.DepthOfBook.Aspen.v1.11': element 'Message' (packet.message) has Count, Payload(Buffer=Rest), and runtime Branch rules but no Size rule/dependency. ZeroCopy length-prefixed packet reader generation requ...

## §67

Size rule carries more than one Operator parameter

first error: Size rule has 2 Operator parameters; at most one is valid for this operation.

## §57

CS0101 the namespace already contains a definition for the type

first error: error CS0101: The namespace 'Cboe.C1Options.BinaryOrderEntry.Boe3' already contains a definition for 'LegPositionEffect'

## §58

CS0104 'Exception' is an ambiguous reference

first error: error CS0104: 'Exception' is an ambiguous reference between 'Hkex.HkexSecurities.Index.Omd.Exception' and 'System.Exception'

## §68

CS0102 duplicate 'PartiesGroupList' emission

first error: error CS0102: The type 'SequencedMessage' already contains a definition for 'PartiesGroupList'

## §69

CS0103 'MessageHeader' does not exist in the RoundTrip harness

first error: error CS0103: The name 'MessageHeader' does not exist in the current context

## classes-enum-nonnumeric-ascii-value

generator defect: integer-enum member with single-character ASCII data crashes generation with a FormatException

first error: The input string 'T' was not in a correct format. Classes side of the same model shape as ZeroCopy's `enum-nonnumeric-ascii-value` (see that key): NoUnspecifiedUnitReplay is a 1-byte Integer-translated field whose PDF-documented values mix numeric bytes (False=0, True=1) with ASCII-letter sentinels (Test=T, and for the v2.3.7 variant also U/E/A). `Scaled.CSharp.Classes.CSharp.Enum.CanonicalKey.For` (Classes/Scaled.CSharp.Classes/CSharp/Enum/CanonicalKey.cs:17) only checks `Is.Ascii.Character(element)` — the whole field's Translation trait — not the individual value's. Since the field is Integer, not Ascii, it falls through to `IntegerValue.For` (Classes/Scaled.CSharp.Classes/CSharp/Enum/IntegerValue.cs, which also does no numeric validation and returns 'T' verbatim) and then `ParseRawUlong.For`, which calls the equivalent of `ulong.Parse("T")` and throws. Fix site: CanonicalKey.For and the enum literal emission path need a per-value ascii-character check (mirroring ZeroCopy's `Literal.Integer`/`Is.Ascii.Character(value)`), not only a per-field one. Covers: Cboe.BzxEquities.BinaryOrderEntry.Boe.v2.3, Cboe.BzxOptions.BinaryOrderEntry.Boe.v2.10, Cboe.C1Options.BinaryOrderEntry.Boe.v2.10, Cboe.C2Options.BinaryOrderEntry.Boe.v2.10, Cboe.CboeEquities.BinaryOrderEntry.Boe.v2.3.7, Cboe.EdgxEquities.BinaryOrderEntry.Boe.v2.3, Cboe.EdgxOptions.BinaryOrderEntry.Boe.v2.10.

## composite-timestamp-no-endian

model defect: a Composite-rule emergent timestamp field carries no Endian trait of its own

first error: Field 'PriorDayTradeDateAndTime' must have exactly one Endian trait. Little=False, Big=False. Fix the model/compiler upstream. `Siac.Cts.Output.Cta.v2.11.Hft`/`.b` declare 'Prior Day Trade Date And Time' (and its Original/Corrected siblings) as `<Rule><Type>Composite</Type>` in OmiSpecifications/Siac/Common/Edits/Siac.Cta.Timestamps.Edits.xml (around line 351), combining a `seconds` field and a nanosecond remainder with Multiply/Plus operators into one emergent field. The emergent field's compiled model element carries Size/Translation/Signedness/Memory traits but no Endian trait — none of its 5 addresses in Siac.Cts.Output.Cta.v2.11.Hft.binary.model.json (priordaytrademessage, priordaytradecancelerrormessage, fractionalpriordaytradecorrectionmessage, fractionalpriordaytrademessage, fractionalpriordaytradecancelerrormessage) has one. This is the Classes-side manifestation of the same upstream gap as ZeroCopy's `leaf-no-translation-or-endian` key, which already covers the sibling 'Corrected Prior Day Trade Date And Time' field on the same models; propose the compiler backfill an Endian trait onto a Composite-rule emergent element from its component field(s), or the spec declare one explicitly. Covers: Siac.Cts.Output.Cta.v2.11.Hft, Siac.Cts.Output.Cta.v2.11.b.

## iex-tops-tradereport-fixture-newer-version

not a defect: `Packets/Iex/Iex.IexEquities.Tops.IexTp.v1.56/TradeReportMessage.pcap` is a TOPS 1.6-era capture filed under the 1.5 model

first error: `[1] ERROR parsing type 'TradeReportMessage': Specified argument was out of the range of valid values. (Parameter 'length')`, thrown by `MemoryMarshal.Read<TradeReportMessage.Layout>(payload)` in the generated `Test/Program.cs` because the layout is larger than the payload. The model declares `Reserved 4` as the last unconditional field of Trade Report and Trade Break, and that matches the vendor document: `OmiDocuments/Iex/IexEquities/TOPS/IEX TOPS Specification (2016).pdf` (TOPS 1.54) lists `(Reserved) 38 4 Reserved bytes` and "Total Message Data length is 42 bytes" for both messages. The field entered the spec in OmiSpecifications commit `8c7a5004` (Update bitfield traits), which also removed the 1.6-only messages from v1.56; that commit is the model hash change between runs. The capture's own Message Length byte reads 38 and its bytes end after Trade ID, which is the TOPS 1.6 layout (v1.64 and v1.66 declare no Reserved field). Same curation problem as `iex-tops-misfiled-auction-fixture`: the v1.56 packets folder holds captures from a later TOPS. Blocked on replacing or removing the fixture. Covers Iex.IexEquities.Tops.IexTp.v1.56 (packets stage only; generation and build pass).

## opaque-field-no-data-translation

model defect: an intentionally opaque byte field carries no Translation trait at all, so it matches none of the generator's type archetypes

first error: Specified argument was out of the range of valid values. (Parameter 'Unknown Type: [Field] Checksum') `ZeroCopy/Scaled.CSharp.ZeroCopy/Files/Type/Elements.cs:11` only recognizes an element as raw data via `Is.Raw.Data`, which requires an explicit `Translation=Data` trait (`ScaledGenerators/Binary/Scaled.Binary.Model.Operations/Traits/Encodings/Translation/Data.cs:18`, `element.HasTrait("Translation", "Data")`); with no Translation trait at all the element also fails `Is.String.Type` and `Is.Integer.Type`, and falls through to the catch-all throw at `Elements.cs:23`. `OmiSpecifications/Nse/Common/Headers/Nnf/Nse.Nnf.DirectTcp.v1.0.Source.xml:253-265` declares the `Checksum` `<Type>` with only `Size` and `Memory` traits; its own file comment (`:183-185`) says the field "is opaque, holding an Md5 digest on the first packet and on non encrypting connections and a Gcm authentication tag otherwise" — the author's intent is raw uninterpreted bytes, but the declaration never adds the `Translation=Data` trait `Is.Raw.Data` looks for, unlike a true raw-data field elsewhere in the corpus. Propose only: add `<Trait><Category>Translation</Category><Value>Data</Value></Trait>` to the `Checksum` `<Type>` block. This surfaced only after the prior defect on this model (`rule-type-complex-unmapped`, "Requested value 'Complex' was not found.") was fixed upstream; one model masking a second. Covers Nse.NseFo.OrderEntry.NnfDirect.v9.50.

## loader-quote-strip-overcounts-whitespace-value

loader defect: quote-stripping normalization leaves stray transcribed whitespace in an all-whitespace enum value, overcounting it past the field's byte width

first error: Character enum 'Quote Condition' value 'Regular Quoteautox Eligible' (or 'Regular Quote') has data '   ' (or '  '). A one-byte ASCII enum value must contain exactly one character. Not a model or spec-transcription defect: the vendor genuinely defines a blank/space byte as a meaningful enum value ("regular quote", the no-condition-set default) — confirmed at `OmiSpecifications/Nasdaq/NomOptions/Nasdaq.NomOptions.Bono.Itch.v3.2.pdf.xml:701` (`<Data>" "  </Data>`, straight quotes), `OmiSpecifications/Nasdaq/PhlxOptions/TopOfMarket/Nasdaq.PhlxOptions.TopOfMarket.Itch.v3.3.Pdf.xml:711` (same shape), and `OmiSpecifications/Nasdaq/NtxOptions/TopOfMarket/Nasdaq.NtxOptions.TopOfMarket.Itch.v1.2.Pdf.xml:707` (`<Data>“ ” </Data>`, curly quotes) — each field is declared `Size=1` and the quoted content is exactly one space, matching the field's byte width. The corruption happens in the shared normalizer: `ScaledGenerators/Builders/Scaled.Text/Text/RemoveQuotes.cs:11-26` (`RemoveQuotesIn`) strips a quoted value by deleting the first and last *occurrence of the quote character anywhere in the string*, not the substring strictly between the matching quote marks; each Data element in this vendor's transcription carries 1-2 trailing spaces after the closing quote (visual column alignment, present on nearly every enum row, e.g. `<Data>"R"  </Data>`). For a non-whitespace value this trailing junk survives quote-stripping but is silently removed by the `.Trim()` in `ScaledGenerators/Builders/Scaled.Text/Text/CheckSpaces.cs:15` (`CheckWhitespaceValueIn`, the non-whitespace branch). For an all-whitespace value, `CheckWhitespaceValueIn:11-13` deliberately skips that same `Trim()` — by design, per its own comment, an all-whitespace result's length is meant to equal the field's byte width — so the trailing junk that quote-stripping left behind is preserved as if it were part of the value: Bono/Phlx (`" "  `, 2 trailing spaces after the closing straight quote) yields 3 spaces; Ntx (`“ ” `, 1 trailing space after the closing curly quote) yields 2 spaces. Propose only: `RemoveQuotesIn` should extract the substring strictly between the first and last matching quote delimiter instead of deleting one occurrence of the character from each end, so trailing whitespace outside the quotes is discarded the same way it already is for non-whitespace values. Covers Nasdaq.NomOptions.Bono.Itch.v3.2, Nasdaq.PhlxOptions.TopOfMarket.Itch.v3.3, Nasdaq.NtxOptions.TopOfMarket.Itch.v1.2 (ZeroCopy); the same loader defect should also surface on the Classes side for these three once/if Classes reaches this field.

## zc-dispatch-multibyte-ascii-discriminator

generator defect: the direct-discriminator check only accepts an integer or single-character ASCII field, not a multi-byte justified/filled ASCII mnemonic

first error: Dispatch 'packet.messagebody' has an unsupported direct discriminator type or width. Same gap as Classes' `classes-dispatch-multibyte-ascii-discriminator`, on the ZeroCopy side: `ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Framing/Branch.cs:70-71` requires the direct discriminator element to satisfy `Is.Integer.Type` or `Is.Ascii.Character` (a single unpadded ASCII byte); `Tmx.Mx.SolaMulticast.Hsvf.v2.1`'s `packet.messagebody` discriminator is a multi-byte justified/filled ASCII mnemonic field, which satisfies neither. Not a model defect — the field's traits (Ascii + Justified + Fill) are unremarkable. Fix site: `ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Framing/Branch.cs:70-71` needs to also accept a multi-byte justified/filled Ascii discriminator. This row previously carried the unrelated key `ascii-no-filled-trait` by mistake (no FileForText/Filled-trait error is involved); re-keyed here. Covers Tmx.Mx.SolaMulticast.Hsvf.v2.1 (formerly Tmx.Mx.Sola.Multicast.Hsvf.v2.1, renamed upstream 2026-09-21).

## zc-counted-group-no-row-cursor

generator defect: ZeroCopy walker does not yet support this counted group's row shape

first error: the generator does not emit counted group 'X' because the group does not have a supported row cursor. `ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Group/Sites.cs:33` (`Sites.UnsupportedReason`) names the first counted group in wire order whose `RowCursor.IsSupported` is false, or (same key, second branch) one that nests another counted group. Before this slice these groups produced an `Info` line and a `// TODO(C)` stub; the generator now fails generation instead of emitting partial coverage. Largest single bucket of the fail-loud slice: every Eurex T7 ETI version, the Ice iMpact versions, and most SBE-sourced messages with repeating groups land here. Fix site: `ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Group/RowCursor.cs` (`IsSupported`) needs a wider row-cursor shape, or `CountedGroups.HasNestedCounted` needs nested-group emission; both are generator capability gaps, not model defects.

## zc-walker-unsupported-path

generator defect: no ZeroCopy walker exists for this tree's shape

first error: no ZeroCopy walker for tree 'X': no path from its root to a message dispatch (or, same `Support.Reject` family: outer dispatch has a key the walker cannot read; loop step has no derivable byte bound; loop step states no Size rule so an outer dispatch inside it cannot skip one sized unit). Fix sites: `ZeroCopy/Scaled.CSharp.ZeroCopy/Source/Generation.cs:41` (no-path case) and `ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Framing/Support.cs:47-52` (`Reject`, the outer-key/bound/count-steps reasons; the bound reason nests a `CountOnly` message from `CountOnly.cs:42,86,184` — no Size/Optional rule to derive consumed bytes, or a Size rule with no dependency field). Header-only trees (Amd/Atp TcpHeader/UdpHeader), pure GapDetection trees, and transport-prefix trees with no reachable dispatch all land here. Before this slice these produced an `Info` line; the walker now fails loud instead of silently emitting no walker. Generator capability gap, not a model defect — covers the task's "walker limits" bucket.

## zc-writer-nonstatic-prefix-child

writer defect: a message's on-path prefix contains a non-static child, so the writer cannot lay out a fixed prefix before the message's variable region

first error: the generator does not emit a writer for 'X' because child 'X' is not static. Fix site: `ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Framing/Writable.cs:163` (`Blocker`, the not-static clause) and `ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Framing/CountOnly.cs:84,111` (the same fact from the count-only path: "a child before the variable region is not static" / "child before the Optional appendages is not static"). Writer-only gap: the ZeroCopy parser (walker) handles these fields; only the writer's prefix-layout requirement rejects them. Attribute to the writer, not the model.

## zc-writer-default-route-no-stamp

writer defect: an outer dispatch's default case has no wire value the writer can stamp to select it

first error: dispatch 'X' routes the walk through its default case and no stamp selects that route. Fix site: `ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Framing/Writable.cs:108` (`AssertTree`, `KeyStampedInEnd` clause). Same shape as the Nyse Pillar `packet.messages` default-route case named in the task brief: the parser's default case is a legitimate no-dependency branch, but the writer has no wire value to write when constructing that case's frame. Writer feature gap (a caller-supplied key value for a default on-path route), not a model defect — matches `output/cleanup-stage5-drcode-2.md`'s follow-up 10 ("Writer feature, not a defect").

## zc-empty-packet-case-not-zero-byte

walker defect: an empty-packet dispatch case's target is not a zero-byte static target, so the empty-packet method the composer selects for it is wrong

first error: no empty-packet method for case 'X' of tree 'X': case target 'X' is not a zero-byte static target. Fix site: `ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Framing/Composer.cs:158` (`EmptyPacketCases`). This is the walker session-packet gap named in the task brief: the case target actually carries bytes of its own (for example the Memx Memoir session-message cases), so treating it as an empty packet is wrong; `docs/zerocopy-session-packets` is the slice that will teach the walker to frame a non-empty case here instead of routing it through the empty-packet path. Blocked on that slice, not a model defect.

## zc-writer-off-path-case-in-loop

writer defect: a dispatch inside the loop step has an off-path case the writer emits no unit for

first error: dispatch 'X' has an off-path case 'X' inside the loop step; the writer emits no unit for it. Fix site: `ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Framing/Writable.cs:111` (and the default-case sibling at `:117`, "off-path default case ... no wire value selects it"). Same shape as the Miax MACH heartbeat-inside-the-loop case named in the task brief: the case is reachable by the parser but the writer has no per-unit emission for an off-path case nested in a loop step. Writer feature gap, not a model defect.

## zc-empty-packet-offset-mismatch

walker defect: the outer dispatch selecting an empty-packet case does not begin where the walk enters the loop step

first error: no empty-packet method for case 'X' of tree 'X': outer dispatch 'X' does not begin where the walk enters the loop step. Fix site: `ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Framing/Composer.cs:161` (`EmptyPacketCases`, the offset clause; contract documented at `:138`). This is the "empty-packet offset shortcut" named in the task brief: the composer's empty-packet shortcut assumed the outer dispatch and the loop-step entry coincide, and this model's outer dispatch begins at a different offset. Walker gap, not a model defect.

## zc-writer-default-branch-no-wire-value

writer defect: a default Branch case targets a message with no wire value the writer can use to select it

first error: dispatch 'X' has a default Branch case targeting message 'X'; no wire value selects a default case, so the writer cannot route to it. Fix site: `ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Framing/Route.cs:45`. Distinct from the resolved `default-branch-case` key (that one is a parser-side rejection removed in the flat-dispatch-rejections slice): this is the writer's inability to construct a frame for a legitimate default Branch case, because writing one requires a value on the wire that identifies it and none exists. Writer feature gap, not a model defect.

## zc-writer-optional-rule-leaf

writer defect: a message's on-path prefix contains a leaf carrying an Optional rule, which the writer's fixed-prefix layout cannot place

first error: the generator does not emit a writer for 'X' because child 'X' carries an Optional rule. Fix site: `ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Framing/Writable.cs:147` (`Blocker`). Same family as `zc-writer-nonstatic-prefix-child` (adjacent clauses in the same `Blocker` switch) but for an Optional rule instead of a non-static child; for example Nasdaq OUCH v5.0's `AccountQueryMessage.AppendageLength` named in the task brief. Writer-only gap, not a model defect.

## zc-writer-size-rule-not-last-child

writer defect: a child carrying a Size rule is not the message's last child, so the writer's fixed-prefix layout cannot place it

first error: the generator does not emit a writer for 'X' because child 'X' carries a Size rule but is not the last child. Fix site: `ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Framing/Writable.cs:159` (`Blocker`, adjacent to the Payload-rule clause at `:157`). Writer-only gap, not a model defect.

## zc-writer-optional-dependency-layout-omits

writer defect: an Optional-ruled field whose rule carries a Dependency, or a trailing Optional-run member, is one the emitted Layout struct excludes, so the writer has no field to stamp the present form through

first error: child 'X' carries an Optional rule with a dependency, so 'X''s layout omits it and the writer cannot return its present form (or, same clause: ''X''s layout omits child 'X', so the writer cannot return its present form' when the Optional rule carries no dependency). Fix site: `ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Framing/Writable.cs:238` (`LayoutExclusionBlocker`) and `:253` (`LayoutExclusionMessage`). The writer-regions slice's rule 3/4 present-form logic (`Blocker`) accepts these fields, but the Layout emitter (`LayoutFields.StructFields`) independently treats a Dependency-bearing Optional as type-dynamic and excludes it from the struct, so the writer has no member to write through even though it would otherwise permit the present form. Covers Jse `newordermessage`/`executionreportmessage`'s `selftradepreventionkey`, Jse Mitch `symboldirectorymessage`'s `leg1symbol`, and Txse `tradingsessionstatusmessage`'s `tradingsessionstatusoperationalhaltreason` (Bale and Feed). Writer/Layout-emitter gap, not a model defect; the follow-up slice ("option B") is to widen the Layout struct to cover the present form for these fields instead of excluding them.

## zc-writer-optional-tail-conflict

writer defect: two trailing Optional-ruled fields read one dependency with different parameters, so no single present form can satisfy both

first error: children 'X' and 'X' both carry Optional rules on 'X' with different parameters. Fix site: `ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Framing/Writable.cs:224` (`OptionalBlocker`, `OptionalTailConflict` clause). Siac Cqs `longquotemessage`'s `nationalbestbidlongappendage` and `nationalbestofferlongappendage` each carry an `Optional` rule on `nationalbboindicator` with a different `Data` value; per the writer-regions plan's rule 4, at most one can be present on the wire, so the writer correctly refuses to pick one for the caller. Writer feature gap (the writer would need a caller-selected case among the mutually exclusive trailing members), not a model defect.

## zc-writer-region-not-trailing

writer defect: a region's own Size rule reads a route stored length, but the region does not end where the unit ends, so the writer cannot bound it

first error: element 'X' carries a Size rule but does not end where the unit ends. Fix site: `ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Framing/Writable.cs:189` (`Blocker`, `LengthBindings.NotTrailing` clause). Coinbase `gapfillmessage`'s `padding` field is exactly the case the writer-regions plan named out of scope: the loop step `sbemessage` has its own trailing `padding` sibling after the `payload` dispatch, so the leaf's region ends before the unit does, and the marker `padding`'s Optional rule also carries an `Index` parameter the corpus spells both `before` and `Before` with no reader on `main`. Blocked on a model/loader question (an `Index` parameter with a stated semantics) before a writer rule can apply; not a defect this slice can fix.

## zc-reassemble-length-dependency-not-one

model or upstream defect: a Reassemble rule states a length dependency count other than one

first error: step 'X' states a Reassemble rule with a length dependency count other than one. Fix site: `ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Framing/Reassemble.cs:49`. One of the task brief's "Reassemble/probe gaps." A Reassemble rule is defined to read exactly one length dependency; a model whose compiled rule names zero or more than one needs the upstream fact traced (ScaledCompilers rule-building for this step) before this can be classified further as model or compiler defect.

## zc-size-rule-dispatch-targets-disagree

model defect: an outer dispatch's targets do not agree on carrying a Size rule

first error: dispatch targets of 'X' do not agree on a Size rule: some carry a Size rule and some do not, once targets whose Size rule only restates the walk bound are treated as unsized. Fix site: `ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Framing/RootBlockLength.cs:26` (`AssertConsistent`). Same clause narrowed by commit `1f16e65` ("Exclude dispatch targets whose Size rule restates the walk bound..."); this row is the residual case that still disagrees after that exclusion. Size-rule contradiction in the task brief's taxonomy — needs the specific model's dispatch targets traced against their Size rules in the compiled model before a spec-level fix site can be named.

## zc-size-rule-exclusion-mismatch

model defect: two length rules on the same field disagree on how many bytes they exclude from the frame size

first error: length rules on 'X' and 'X' both read 'X' but exclude N and N (or N and -N) bytes from the frame size. Fix site: `ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Framing/LengthBindings.cs:110` (`AssertConsistent`, the exclusion-mismatch clause). Size-rule contradiction in the task brief's taxonomy: two rules derived from the same field disagree on the excluded-byte count, which traces to the compiled Size/Payload rules for that field. Needs the specific model's rule pair traced upstream (`ScaledCompilers`) before a spec-level fix site can be named.

## memx-memo-missing-message-session

model defect (not blocking): Memx.MemxEquities.Memo.Sbe session-frame targets carry no `Message=Session` characteristic

The compiled model gives the Memx `MemoirDepthFeed`/`MemoirLastSale`/`MemoirTopOfBook` and Nasdaq `SoupBin` session-frame targets (Heartbeat, Login, Logout, and similar) an action with `characteristics: [{"Message": "Session"}]` (5 in each Memoir model, 15 in SoupBin). The equivalent `Memx.MemxEquities.Memo.Sbe.*` session-frame targets (for example `clientpacket.clientdata.loginrequestmessage`, address `Login Request Message`) carry no `actions` at all, so the characteristic is absent. This is an inconsistency between sibling protocol specifications, not a fact any generator is entitled to require: the ZeroCopy session-packet dispatch (`ZeroCopy/Scaled.CSharp.ZeroCopy/CSharp/Framing/PacketDispatch.cs`) selects session-frame targets from the packet dispatch's off-path Branch cases alone and does not read `Message=Session` at all, so every `Memx.MemxEquities.Memo.Sbe.*` model still generates and passes. Fix site (propose only, not applied): `OmiSpecifications` — the Memx `Memo.Sbe` declarations' session message elements (Login Request/Accepted/Rejected, Logout, Heartbeat) would need the same `<Insert>`/characteristic declaration the Memoir and SoupBin declarations carry, once an owner confirms the omission is a transcription gap rather than an intentional difference between the two Memx protocols.

## rule-type-complex-unmapped

generator defect: the shared model loader cannot parse a `Complex` rule type the compiled model carries

first error: Requested value 'Complex' was not found. `ScaledGenerators/Binary/Scaled.Binary.Model.Reader/Load/Json/Rules.cs:18` calls `Enum.Parse<RuleType>(...)` on every rule's `type` string with no fallback; `ScaledGenerators/Binary/Scaled.Binary.Model/Definitions/Enums/RuleType.cs` has no `Complex` member, so loading throws before any target-specific generator runs. All 54 affected models are Nse (NseCd/NseCm/NseCom/NseFo Mtbt/MtbtNdal/Recovery/Snapshot/Binary and the NnfBcast/Nnf/NnfDirect/NnfTrimmed order-entry models); each carries at least one field whose `rules` array holds an entry with `"type": "Complex"` and its own nested traits (for example `Nse.NseCd.Mtbt.Binary.v6.8`'s `Order Id` field, address `packet.message.payload.newordermessage.orderid`, traits `Translation=FloatingPoint`, carries a rule `{"type": "Complex", "traits": [{"Size": "8"}, {"Translation": "Integer"}, {"Signedness": "Unsigned"}]}`). This loader gap masks the pre-existing `floating-point-field` defect on the same fields (previously the first error reported was the FloatingPoint-translation gap in ZeroCopy; now loading fails earlier, before that field-level classification runs). Not caused by the ZeroCopy stream-assembly slice: the throw site is in `ScaledGenerators`, shared model-loading infrastructure outside CSharpGenerators, and the affected models are unrelated to the slice's Reassemble/stream/session-packet scope. Needs `ScaledGenerators` owner attention to add a `Complex` case (or an explicit unsupported-type message) to `RuleType` and its consumers.

## classes-dispatch-session-action-no-class

generator defect: the root dispatch switch calls `.Parse` on session-action message targets that never get a generated class

first error: The name 'ServerHeartbeat' does not exist in the current context (also seen as `AdminHeartbeat`, `ServerHeartbeatPacket`, `HeartbeatMessage`, `ServerHeartbeatMessage`, `ClientHeartbeatMessage`, each in that model's `Framing/Dispatch.cs`). A session-control target such as Asx's `ServerHeartbeat` carries the `Message` characteristic (`Is.Message.Type` true, so `Dispatch/Validation.cs:29`'s "neither a generated message nor a branch container" guard does not fire) but its element `Type` is `ElementType.Action`, not `ElementType.Group`, so `ClassesFiles.cs:53`'s `context.Model.Structs(Is.Message.Type)` — which first filters to `Is.Struct` (`ElementType.Group`) — never emits a class file for it. `Files/Packet/Components/DispatchSwitchCases/List.cs` and `DispatchCase.cs:15` build every case through `CSharp/Dispatch/ParseExpression.cs`, which always emits `{Declaration}.Parse(payload)` with no check for `Is.Message.Action`/`Is.Empty`, so the case references a class that was never generated. `EmptyDispatchPredicateConstants/List.cs` only recognizes `Is.Empty` cases, so `IsKnownEmptyType` also stays `false` for these targets — confirmed in the generated Asx and Nasdaq SoupBin output, where `IsKnownEmptyType` returns `return false;` with no names even though `ServerHeartbeat`/`ServerHeartbeatPacket` are session-control targets. Fix site: `Classes/Scaled.CSharp.Classes/CSharp/Dispatch/ParseExpression.cs` (and/or `Dispatch/Validation.cs`'s message-target check) needs a case for `Is.Message.Action` targets, parallel to the existing `Is.Empty` handling. Covers 52 models across Asx, Bist, Biva, Cme (Globex Mdp3/Settlements/Streamlined), Iex, Jnx, Jpx, Nasdaq (Common SoupBin, Gemx/Ise/Mrx/Ntx/Phlx Options Glimpse, Nsm/Ntx/Psx Orders Ouch, Ntx TotalView Glimpse, Utp Snapshot), NsxAustralia, Odx, Tadawul.

## classes-roundtrip-header-selection-mismatch

generator defect: the RoundTrip padding-test selector emits a test for every `Is.Header` struct in the model, not only the one header file that `HeaderFiles.For` actually generates

first error: The name 'MessageHeader' does not exist in the current context (`Siac.GapDetection.Cta.v3/RoundTrip/Program.cs`). `Siac.GapDetection.Cta.v3` carries two `Is.Header` Group elements: `packet.blockheader` (`Header=Packet`) and `packet.message.messageheader` (`Header=Message`). `Files/Header/HeaderFiles.cs:11` emits exactly one header class, the single resolved `generatedPacketHeader` (here `BlockHeader.cs`); it never emits a file for a message-level header. `Files/Test/Components/RoundTrip/TestSelection/List.cs:14` instead iterates `context.Model.Structs(Is.Header)` — every header in the tree — and calls `RoundTrip.List.ForHeader` for each, so it also emits a padding-clear test for `packet.message.messageheader`, whose declaration name (`CSharp.Message.ClassDeclaration.For`) resolves to `MessageHeader`, a class that was never generated. Fix site: `Classes/Scaled.CSharp.Classes/Files/Test/Components/RoundTrip/TestSelection/List.cs:14` should select from the same header set `HeaderFiles.For` emits, not every `Is.Header` struct in the model. Covers Siac.GapDetection.Cta.v3 at the build stage; `Siac.Cqs.Input.Cta.v2.9.b` and `Siac.Cts.Input.Cta.v2.7.f` carry the same two-header shape but now fail earlier, at generation, under `classes-nested-dispatch-empty-target-unsupported`.

## classes-bitflag-name-collision-clear

generator defect: a bit-flag property named directly from its model value collides with the base `Field.Clear()` lifecycle method

first error: 'AdapFlagsField.Clear' hides inherited member 'Field.Clear()'. Use the new keyword if hiding was intended. The Cboe Pitch `AdapFlags` bitfield carries a one-bit child literally named "Clear". `Classes/Scaled.CSharp.Classes/Files/Field/Components/Value/BitfieldChildValue/BitfieldFlagValue.cs:8` sets `Name => Format.PascalCase(Element.Name)` with no check against reserved member names on the field's base class, so it emits `public bool Clear => (Value & ClearMask) != 0;` in the same class that already declares `public override void Clear()` from the field lifecycle. The property does not throw a hard conflict (a property and a zero-arg method can share a name in C# when one hides the other) but does trigger CS0108, and the generator's builds treat warnings as errors. Fix site: `Classes/Scaled.CSharp.Classes/Files/Field/Components/Value/BitfieldChildValue/BitfieldFlagValue.cs:8` needs to qualify or rename a flag name that collides with a `Field` base member (`Clear`, and any other reserved lifecycle name). Covers Cboe.ByxEquities.SummaryDepth.Pitch.v1.0.4, Cboe.BzxEquities.SummaryDepth.Pitch.v1.0.4, Cboe.EdgaEquities.SummaryDepth.Pitch.v1.0.4, Cboe.EdgxEquities.SummaryDepth.Pitch.v1.0.4; all four previously failed generation under §38, which this run fixed.

## classes-roundtrip-catch-unqualified-exception

generator defect: the RoundTrip failure-catch block writes a bare `catch (Exception ex)` that collides with a model-defined type also named `Exception`

first error: 'Exception' is an ambiguous reference between 'Hkex.HkexSecurities.Index.Omd.Exception' and 'System.Exception'. The Hkex Omd models generate an enum named `Exception` (`Enums/Exception.cs`) in the same namespace as the generated `RoundTrip/Program.cs`. `Classes/Scaled.CSharp.Classes/Files/Test/Components/RoundTrip/RoundTripFailureCatch.cs:11` hardcodes `catch (Exception ex)`, which the compiler cannot resolve unambiguously once a sibling type shares that name. Fix site: `RoundTripFailureCatch.cs:11` should qualify the catch as `System.Exception`. Covers Hkex.HkexSecurities.Index.Omd.v1.44/v1.45, Hkex.HkexSecurities.IndexRefresh.Omd.v1.44/v1.45, Hkex.HkexSecurities.IndexRetrans.Omd.v1.44/v1.45; all six previously passed.

## classes-reassemble-bound-unsupported-shape

generator defect: `Reassembly.Require` only accepts a Reassemble rule whose target is exactly the walk's per-message loop bound, and `Bound.For` only derives that bound from a `Size` rule; two other valid wire shapes fail loudly

first error: Model 'Lseg.Lse.Level1Recovery.Gtp.v26.2': Reassemble on 'packet' requires the walk's message bound, but the bound is 'packet.message'. Also seen as "...but the bound is 'none'." and "...must depend on the Size dependency '...'." — all three are different clauses of the same `Classes/Scaled.CSharp.Classes/CSharp/Framing/Reassembly.cs:19-30` check. Two distinct wire shapes hit it: (1) Lseg (23 models) and OtcMarkets (3 models) declare `Size` and `Reassemble` on the outer `packet` element, which itself carries a repeated, individually `Size`-ruled `packet.message` child (`Count` on `packet.unitheader.messagecount`, `Size` on `packet.message.messageheader.messagelength`) — one length-prefixed TCP/UDP packet holds N self-lengthed messages. `Bound.For` (`CSharp/Framing/Bound.cs:33-40`) walks to the loop step (`packet.message`) and finds its own `Size` rule, so it reports the bound as `packet.message`, which then fails to match `reassemble.Address` (`packet`). (2) The 20 Memx `Memo.Sbe`/`RiskControl.Sbe` models and `Memx.MemxEquities.CommonHeader.Tcp.v1.2` declare `Reassemble` directly on the packet root with its own `Dependency` parameter and no companion `Size` rule anywhere in the tree (single message per packet, no repetition); `Bound.For` only ever trusts a `Size` rule, so `SizedBelowLoop` returns null and the bound is reported as `none`. `Nasdaq.Common.Xmp.Tcp.v1.0` hits the second clause of the same check: its `Reassemble` dependency (`packet.xmppacket.packetheader.bodylength`) does not match the `Size` dependency `Locate.Size.Dependency` resolves for the bound it does find. None of these are model defects — each wire shape is a real, unremarkable framing pattern (packet-level length prefix around repeated self-lengthed messages; a single Reassemble-only frame with no inner repetition) that the newly rule-driven walker does not yet cover. Fix site: `Classes/Scaled.CSharp.Classes/CSharp/Framing/Reassembly.cs` and `Bound.cs` need to accept a Reassemble target above the loop step when the loop step's own Size rule nests inside it, and to derive a bound directly from a Reassemble rule's own `Dependency` parameter when no separate `Size` rule exists. Covers 47 models: 23 Lseg (Lse Level1/Level2 Mbo/Mbp/Incremental Recovery+Replay, Mifid2PostTrade Recovery+Replay, TradeEcho Level2Incremental/Mifid2PostTrade Recovery+Replay, Turquoise Recovery+Replay), 3 OtcMarkets (LinkNqb/MoonAts/Overnight Retransmission), 20 Memx (MemxEquities CommonHeader.Tcp.v1.2 + Memo.Sbe.v1.1/1.2/1.6/1.8/1.9/1.10/1.11/1.12, MemxOptions Memo.Sbe.v1.3/1.5.b/1.6.a/1.6.b/1.7/1.8/1.9/1.10 + RiskControl.Sbe.v1.3/1.6/1.7), and Nasdaq.Common.Xmp.Tcp.v1.0. A later OmiSpecifications update added a `tcpunit` wrapper element inside the Lseg `packet` tree (model_hash unchanged for the 22 already-covered Lseg rows, since the wrapper is a compiler/loader-side regrouping, not a spec edit visible at this model's hash — confirmed by re-running: same model_hash, error text moved from `Reassemble on 'packet'`/`bound is 'packet.message'` to `Reassemble on 'packet.tcpunit'`/`bound is 'packet.tcpunit.message'`); same `Bound.For`/`Reassembly.Require` mismatch, one level deeper. Also covers the two new Lseg Millennium declarations (`Lseg.Millennium.Level2Recovery.Mitch.v11.9`, `Lseg.Millennium.Level2Replay.Mitch.v11.9`, added to the corpus this run), which hit the identical `packet.tcpunit`/`packet.tcpunit.message` shape.

## classes-countedgroupstate-duplicate-group-name

generator defect: counted-group state (property + backing fields) is emitted once per raw group element instead of once per distinct container name, so two sibling dispatch-case groups that share a name collide

first error: Model 'Nasdaq.Utp.Input.Utp.v4.0': `Messages/Application/UnsequencedDataPacket.cs` declares `public OddLotBidShortFormAttachmentList? OddLotBidShortFormAttachmentList { get; set; }` twice (build error CS0102). `CSharp.Group.UnbranchedIn.For` (`Classes/Scaled.CSharp.Classes/CSharp/Group/UnbranchedIn.cs:30`) descends through dynamic (dispatch) boundaries and collects every homogeneous counted group reachable in the subtree, including groups that live in different, mutually exclusive nested-dispatch case branches — here `Exchange Odd Lot Quote Message Short Form Message` and `Exchange Combined Quote Message Short Form Message`, two sibling cases under `inboundquotemessagesmessagepayload`, each declare their own `Odd Lot Bid Short Form Attachment` / `Odd Lot Ask Short Form Attachment` group. `CSharp.Group.ContainerTypeNames.For` (`ContainerTypeNames.cs:10`) and `CSharp.Group.DistinctGroups.For` (`DistinctGroups.cs:10`) both dedupe by container type name for exactly this reason ("one per distinct container type name... callers that render one member per group"), and `Files/Message/StructFile.cs:58` already computes `DistinctGroups = CSharp.Group.DistinctGroups.For(groups, context)`. But `StructFile.cs:91` passes the raw, non-deduped `Groups` into `Components.CountedGroupState.List.For(Groups)`, so `CountedGroupState.Source` (`Files/Message/Components/CountedGroupState/CountedGroupState.cs:16`) emits one property/state block per group *element*, not per distinct name, and the two same-named groups collide. Fix site: `Files/Message/StructFile.cs:91`, change `Components.CountedGroupState.List.For(Groups)` to `Components.CountedGroupState.List.For(DistinctGroups)`. Generator defect, not a model defect — the model's two sibling dispatch cases legitimately declare identically-named inner groups. This model previously failed generation under `§52` (a since-fixed Justified-trait gap); fixing that unmasked this build-stage collision. Covers Nasdaq.Utp.Input.Utp.v4.0.

## tmx-startofframe-missing-signedness

model defect: the Tmx Xmt header's "Start of Frame" field declares Size/Translation/Memory but no Signedness trait

first error: Element 'packet.frameheader.startofframe' has no supported enum underlying type. (Parameter 'element') `Classes/Scaled.CSharp.Classes/CSharp/Enum/Type.cs` walks `Is.OneByte.SignedInteger`/`Is.OneByte.UnsignedInteger` through `Is.EightByte.*` before throwing; none match because the compiled field carries only `Size=1`, `Translation=Integer`, `Memory=Bytes`, with no `Signedness` trait at all. Confirmed at the source: `OmiSpecifications/Tmx/Common/Headers/Xmt.Header.Udp.v1.1.Source.xml:104-120` declares the `Start of Frame` `<Type>` with `Size`, `Translation`, and `Memory` `<Trait>` blocks only — no `Signedness` trait, unlike its sibling `Protocol Version` type a few lines below. Fix site (propose only, not applied): `OmiSpecifications/Tmx/Common/Headers/Xmt.Header.Udp.v1.1.Source.xml`, add a `<Trait><Category>Signedness</Category><Value>Unsigned</Value></Trait>` to the `Start of Frame` `<Type>` block (the field's one enumerated value, `New Frame`=2, is consistent with an unsigned marker byte, but an owner should confirm against the vendor document before this is applied). Covers all 9 models that include this shared header: Tmx.QuantumFeed.AlphaLevel1.Xmt.v2.1/v2.2, Tmx.QuantumFeed.AlphaLevel2.Xmt.v2.1/v2.2, Tmx.QuantumFeed.TsxTsxvLevel1.Xmt.v2.6/v2.8, Tmx.QuantumFeed.TsxTsxvLevel2.Xmt.v2.1/v3.6, Tmx.QuantumFeed.XmtHeader.Udp.v1.1.

## classes-dispatch-multibyte-ascii-discriminator

generator defect: the direct-discriminator check only accepts an integer or single-character ASCII field, not a multi-byte justified/filled ASCII mnemonic

first error: Dispatch 'packet.messagebody' has an unsupported direct discriminator type or width. `Box.Options.Sola.Multicast.Hsvf.v1.5`'s `packet.messageheader.messagetype` is a 2-byte, left-justified, space-filled ASCII field carrying mixed 1- and 2-character mnemonic codes (`U`, `V`, `C`, `CS`, `FS`, `GC`, ...). `Classes/Scaled.CSharp.Classes/CSharp/Framing/Branch.cs:56-59` requires the direct discriminator element to satisfy `Is.Integer.Type` or `Is.Ascii.Character` (a single unpadded ASCII byte); a justified/filled multi-byte ASCII field satisfies neither, so every dispatch keyed on this field throws regardless of width. This is a real, common wire shape for these Sola HSVF protocols, not a model defect — the field's traits are unremarkable Ascii+Justified+Fill. Fix site: `Classes/Scaled.CSharp.Classes/CSharp/Framing/Branch.cs:56-59` needs to also accept a multi-byte justified/filled Ascii discriminator, not only `Is.Ascii.Character`. Covers Box.Options.Sola.Multicast.Hsvf.v1.5/v1.8/v1.9, Box.Options.Sola.Unicast.Hsvf.v4.5.1, Tmx.Mx.Sola.Multicast.Hsvf.v1.11/v1.13/v1.14/v2.1.

## classes-composite-dispatch-empty-target-unsupported

generator defect: composite-key dispatch explicitly rejects any case whose target is an empty element

first error: Model 'Nasdaq.Uqdf.Output.Utp.v3.0.Hft': composite-key dispatch target 'moldudp64packet.messages.message.udppayload.startofdaymessage' has no body. `Classes/Scaled.CSharp.Classes/Files/Packet/Components/CompositeDispatchChildren/List.cs:16-19` throws `NotSupportedException` by design — its class comment states the composite-key container "has no empty-type predicate" — whenever a case target is `Is.Empty`, because composite-key dispatch (new in this pass) has no equivalent of the root dispatch's `IsKnownEmptyType` mechanism. Fix site: `Classes/Scaled.CSharp.Classes/Files/Packet/Components/CompositeDispatchChildren/List.cs` needs an empty-type predicate for composite-key dispatch, the same feature `classes-nested-dispatch-empty-target-unsupported` needs for nested dispatch. Covers Nasdaq.Uqdf.Output.Utp.v3.0.Hft, Nasdaq.Utdf.Output.Utp.v3.0.Hft (both regressions from passing), and Siac.Cts.Output.Cta.v2.11.Hft (previously §67).

## classes-double-consume-guard-nested-header

generator defect (needs investigation): the double-consume guard assumes no message body ever re-declares its own dispatch-key field, which does not hold for the Nyse Pillar models

first error: Double-consume invariant violated in model 'Nyse.AmexOptions.BinaryGateway.PillarStream.v3.25': message 'Login Message' includes dispatch-key source field (address 'pillarstreammessage.loginmessage.msgheader.msgtype') among its descendants. The caller has already advanced past the header; the message parse constructor must not re-consume a dispatch-key field. `Classes/Scaled.CSharp.Classes/Files/Container/Components/Parsing/DiscriminatorFieldsGuard.cs:20-27` throws for any target whose own descendant tree contains the discriminator field's address. For these three Nyse Pillar models, `Login Message`'s compiled tree does contain `msgheader.msgtype` among its own descendants — the model represents each message's header as a nested child of the message rather than a sibling consumed once before dispatch. Whether that nested-header shape is itself correct (matching the wire) or whether the guard's assumption is too strict for a model that legitimately re-declares its header inline has not been confirmed; this needs a `Classes` generator owner to decide whether the fix is to skip re-parsing the nested header field during the message body walk, or to loosen/scope the guard. Fix site: `Classes/Scaled.CSharp.Classes/Files/Container/Components/Parsing/DiscriminatorFieldsGuard.cs`. Covers Nyse.AmexOptions.BinaryGateway.PillarStream.v3.25/v3.27, Nyse.ArcaOptions.BinaryGateway.PillarStream.v3.25/v3.27, Nyse.Options.StreamProtocol.PillarStream.v1.6; the v3.25 members previously failed generation under §62, which that run fixed, and the v3.27 pair are new declarations added by the 2026-09-21 Nyse upstream update carrying the same nested-header shape.

## classes-nested-dispatch-empty-target-unsupported

generator defect: nested dispatch methods have no empty-type predicate, unlike the root dispatch overload, and explicitly reject an empty case target

first error: Model 'Siac.Cqs.Input.Cta.v2.9.b': nested dispatch 'packet.message.categorypayload.controlmessage.controlmessagepayload' targets empty element 'packet.message.categorypayload.controlmessage.controlmessagepayload.startofdaymessage'; a nested dispatch method has no empty-type predicate to serve it. `Classes/Scaled.CSharp.Classes/Files/Packet/Components/NestedDispatchMethods/List.cs:17-21` throws `NotSupportedException` by design for any `Is.Empty` case target inside a nested (container-routed) dispatch; the class comment states this limitation explicitly. Fix site: `Classes/Scaled.CSharp.Classes/Files/Packet/Components/NestedDispatchMethods/List.cs` needs the same empty-type predicate the root dispatch overload has via `Files/Packet/Components/EmptyDispatchPredicate.cs`. Covers Siac.Cqs.Input.Cta.v2.9.b, Siac.Cts.Input.Cta.v2.7.f; both previously failed at the build stage under §69 (the same models' `classes-roundtrip-header-selection-mismatch` two-header shape), and now fail earlier, at generation.

## classes-boe-logout-messagetype-collision

model defect (needs confirmation): one Cboe BOE message-type code routes to both an empty client message and a parsed server message

first error: Model 'Cboe.CboeEquities.BinaryOrderEntry.Boe.v2.3.7': one dispatch code targets both empty and parsed elements. `Classes/Scaled.CSharp.Classes/CSharp/Dispatch/Validation.cs:35-37` throws when the same wire value routes to both an empty case and a routed (parsed) case. In this model, `packet.message`'s Branch rules use `Data: '0x02'` twice: once for the empty `packet.message.logoutrequestmessage` (client→server) and once for the non-empty `packet.message.logoutmessage` (server→client, carries a `Logout Reason` field). Both share one dispatch tree under `packet.message` with no direction split, unlike sibling protocols (for example Memx `Memo.Sbe`) that compile client-originated and server-originated messages into two separate root trees. Whether this is a compiler defect (client/server BOE messages should be split into separate trees the way Memx's are) or a legitimate same-value-different-direction wire shape that the Branch model can't represent as one dispatch has not been confirmed with an owner. Fix site (propose only, not applied): `ScaledCompilers`, wherever the packet tree is assembled for `Cboe.CboeEquities.BinaryOrderEntry.Boe.v2.3.7` (or the shared Boe loader), would need to route client-originated and server-originated messages into separate dispatch trees by `Origin`.

## iex-tops-misfiled-auction-fixture

not a defect: `Packets/Iex/Iex.IexEquities.Tops.IexTp.v1.56/AuctionInformationMessage.pcap` captures a message type this model does not define

first error: Packet Iex.IexEquities.Tops.IexTp.v1.56/AuctionInformationMessage.pcap exited 1 (`Packets: 1, Messages: 0, Errors: 1` — the harness never decoded a single message, unlike a per-message parse failure). Decoding the fixture directly: after the shared 40-byte IEXTP header, the sole IEXTP message has on-wire `Message Type` byte `0x41` ('A'). `Iex.IexEquities.Tops.IexTp.v1.56.binary.model.json` has no message, field, or characteristic anywhere in the tree whose name contains "Auction" — TOPS v1.56 does not declare an Auction Information message at all (that message belongs to the IEX DEEP feed, a different protocol). The generated dispatch therefore has no case (and no default case) for type `0x41`, so the walk fails before producing any message. This is a fixture-curation problem, not a model or generator defect: the pcap under this model's `Packets/` folder was captured against, or copied from, a different IEX feed/version than the one it is filed under — consistent with the wider corpus fact (see `project_packets_folder_state` in agent memory) that a large fraction of `Packets/` folders do not actually match their model id. Blocked on: replacing or removing this fixture; needs an owner with access to a genuine TOPS v1.56 Auction-adjacent capture, or confirmation that TOPS v1.56 never carries this message and the fixture should be deleted from this model's folder. The classes-heartbeat-dispatch run fails on a second pcap in the same folder, `OfficialPriceMessage.pcap` (also `Messages: 0, Errors: 1`; Official Price is likewise a DEEP-feed message), byte-for-byte the same shape — the folder holds more than one misfiled fixture, and whichever one the harness reaches first varies run to run. The final rerun of the same slice (`output/verify-classes-binary-n6_39by5`) lands back on `AuctionInformationMessage.pcap`, byte-identical to the first-run log — this fixture's failure is unaffected by the ConstantType/ConstantLiteral generator fix.

## classes-pcap-harness-silent-error

unresolved: a Nyse ArcaEquities pcap fixture fails with no reported error text, cause not yet known

first error: Packet Nyse.ArcaEquities.IntegratedFeed.Pillar.v2.5.g/ReplaceOrderMessage.pcap exited 1. The retained packet-fixture log (`logs/packets/Nyse.ArcaEquities.IntegratedFeed.Pillar.v2.5.g.ReplaceOrderMessage.log`) shows `Packets: 1, Messages: 1, Errors: 1` with no exception text; the generated `Test/Program.cs` harness increments an `errors` counter on a caught exception but never prints `ex.Message` or the exception type, so the cause is not visible from the retained log, and the generated project is not kept after the run to rerun directly. This row was already an unattributed TODO before this run (previously failing on a different pcap in the same folder, `CrossTradeMessage.pcap`, also with no detail), so the pcap that trips it appears to vary run to run rather than being fixed by this pass. The 2026-09-21 rebase run repeats the pattern a third time, on a third different pcap in the same folder (`SecurityStatusMessage.pcap`, `Packets: 1, Messages: 1, Errors: 1`, again no exception text) — the model was not touched by this rebase, only the run's pcap-fixture ordering/selection differs, reinforcing that the trigger is per-fixture, not per-model-change. The classes-heartbeat-dispatch run repeats it a fourth time on a fourth pcap (`ReplaceOrderMessage.pcap`), again not a model this slice's tree diff touched. The final rerun of the same slice (`output/verify-classes-binary-n6_39by5`) lands on a fifth pcap, `SecurityStatusMessage.pcap` (`Packets: 1, Messages: 1, Errors: 1`, no exception text, log byte-identical to the corresponding log in the first run) — same shape, unaffected by the ConstantType/ConstantLiteral generator fix. Blocked on: rerunning `sj verify-classes-all --keep` (or `sj check-classes Nyse.ArcaEquities.IntegratedFeed.Pillar.v2.5.g`) and reading the kept project's Test harness output directly, or adding exception detail to the generated harness's failure branch, to see the actual thrown message.

## classes-pillar-sequenced-filler-message-sized-action

generator gap: a Nyse Pillar "filler" message Action carries a variable-size Size rule (padding to the header's declared message length), a shape `RequireEmptyBody` correctly rejects but the generator has no case to parse

first error: Model 'Nyse.AmexOptions.BinaryGateway.PillarStream.v3.25': message Action 'pillarstreammessage.seqmsg.sequencedmessage.sequencedfillermessage' states a runtime rule; a message Action has an empty body. `pillarstreammessage.seqmsg.sequencedmessage.sequencedfillermessage` has no children and a `Size` rule of `seqmsglength - 4` (confirmed identical in all four affected models): the wire message is a discriminated "filler" whose entire body is padding sized to fill out the declared record length, not a zero-byte Action. `Classes/Scaled.CSharp.Classes/CSharp/Message/Selection.cs:57-72`'s `RequireEmptyBody` throws for any Message Action that states a Size/Count/Payload/branch dependency rule or has children, by design (plan `docs/classes-heartbeat-dispatch/plan.md`: "An Action with payload or an unsupported wire shape must fail clearly"). This is not a defect in that check; it is a real, unsupported wire shape — a sized/skip-body Action — that has no generated representation yet. Before this slice these four models failed earlier at generation under `classes-double-consume-guard-nested-header` (a nested-header double-consume on `Login Message`, elsewhere in the same trees), which masked this filler-message gap; this run's text change from the double-consume error to this one is the expected effect of narrowing `classes-double-consume-guard-nested-header`'s masking, not a regression. Fix site: a sized-filler/raw-skip case in `Classes/Scaled.CSharp.Classes/CSharp/Message/Selection.cs`'s empty-body validation (or a sibling message shape) that reads the Size rule and emits a body that consumes and discards the stated byte count. Covers Nyse.AmexOptions.BinaryGateway.PillarStream.v3.25/v3.27, Nyse.ArcaOptions.BinaryGateway.PillarStream.v3.25/v3.27.

## classes-message-type-constant-literal-bitfield-mismatch

fixed in this slice (classes-heartbeat-dispatch, scotty-4): `ConstantType.For` and `ConstantLiteral.For` classified a Branch discriminator by different predicates, so a 1-bit flag discriminator got a `byte`-typed constant paired with a single-character literal

original error: OtcMarkets.LinkAts.Headers.Link.v1/OtcMarkets.LinkAts.Headers.Link.v1/Messages/Session/HeartbeatPacket.cs(19,37): error CS0266: Cannot implicitly convert type 'char' to 'byte'. `packet.messageblock`'s Branch rule keys off `packet.packetheader.packetflag.heartbeat`, a 1-bit field (`Size: 1`, `Memory: Bits`) inside the `Packet Flag` Composite, tested with `Operator: Equals`, `Data: "1"` — not the usual byte/ASCII "Message Type" field. `ConstantType.For` classified the discriminator by `Is.Ascii.Character` and `discriminator.Bytes()` alone (a 1-bit field's rounded byte width is 1, unsigned), returning `byte`. `ConstantLiteral.For` classified the same discriminator by `Is.Integer.Type` (false — the bit field itself carries no `Translation` trait; `Translation: Integer` sits on the parent Composite rule, not the field) and fell through to the single-ASCII-character path, treating `Data: "1"` as the character `'1'`. The two helpers disagreed on the same element, so `MessageType.cs:17` emitted `public const byte MessageType = '1';`. This model passed at baseline; the `Heartbeat Packet` Action (`Message: Session`) was excluded before this slice and was newly selected, which is why the mismatch became reachable.

fix: one shared discriminator classification (`Classes/Scaled.CSharp.Classes/CSharp/DiscriminatorKind.cs` and `CSharp/Discriminator/Kind.cs`, new enum `Character | Integer | AsciiCode | Bits`) now backs `Discriminator/TypeName.cs`, `Message/ConstantType.cs`, `Message/ConstantLiteral.cs`, `Field/Bitfield.cs` (`MaximumValue`), `Framing/KeyType.cs`, `Framing/Branch.cs` (`Cases`/`Canonical`/`ReadsWholeBytes`/`DispatchKey`), and `Framing/Encodable.cs` (`IsCoveredStatic`). Generated output for the model is now `public const byte MessageType = 1;`. Corpus tree diff confirmed exactly one file changed corpus-wide (`OtcMarkets.LinkAts.Headers.Link.v1/.../HeartbeatPacket.cs`, line 19 only); `OtcMarkets.LinkAts.Headers.Link.v1` returned to `passed` in the `2026-09-22` `verify-classes-all` rerun (`output/verify-classes-binary-n6_39by5`). No row in `classes.tsv` currently carries this key; it remains here as closed history.

# Compile

## §70

case-sensitive Pdf.xml glob misses lowercase pdf.xml source files on Linux

first error: Missing Source: Asx.AsxDerivatives.Ntp.Itch.v1.05.Pdf.xml. Compiler defect, not a model defect. ScaledCompilers/Binary/Scaled.Binary.Specification.Builder/Library/Dictionaries/Sources.cs:79 indexes source files with `Directory.EnumerateFiles(root, "*.Pdf.xml", SearchOption.AllDirectories)`. Every affected declaration's actual OmiSpecifications file is named with a lowercase extension (for example Asx/Ntp/Asx.AsxDerivatives.Ntp.Itch.v1.05.pdf.xml, Nyse/Common/Source/Equities/BinaryGateway/Nyse.Equities.PillarStream.BinaryGateway.v5.17.pdf.xml, and the sibling v5.8 file). .NET's glob matching is case-sensitive on Linux, so the pattern silently excludes them; the previous good run recorded `/Users/sean/...` paths, i.e. macOS, where HFS/APFS matches case-insensitively and hid the bug. Propose changing the pattern to a case-insensitive match (e.g. `EnumerationOptions { MatchCasing = MatchCasing.CaseInsensitive }`) or lowercasing both sides of the comparison; the same risk applies to the other four extension patterns in `FilesIn` (`*.Source.xml`, `*.Exchange.xml`, `*.Xls.xml`, `*.Msgs.Txt`, `*.asn`) if any on-disk file uses different case. Covers all 12 compile.tsv rows: asx.asxderivatives.ntp.itch.v1.05, nyse.amexequities.binarygateway.pillarstream.v5.17, nyse.amexequities.binarygateway.pillarstream.v6.0, nyse.arcaequities.binarygateway.pillarstream.v5.17, nyse.arcaequities.binarygateway.pillarstream.v6.0, nyse.nationalequities.binarygateway.pillarstream.v5.17, nyse.nationalequities.binarygateway.pillarstream.v6.0, nyse.nyseequities.binarygateway.pillarstream.v5.17, nyse.nyseequities.binarygateway.pillarstream.v5.8, nyse.nyseequities.binarygateway.pillarstream.v6.0, nyse.texasequities.binarygateway.pillarstream.v5.17, nyse.texasequities.binarygateway.pillarstream.v6.0. The v6.0 rows are new declarations added by the 2026-09-21 Nyse upstream update; their actual source files (Nyse.Equities.PillarStream.BinaryGateway.v6.0.pdf.xml) are on disk with a lowercase extension, same root cause.

## itch-ascii-pdf-loader-typo

model defect: `<Loader>` names a loader identifier that does not exist

first error: Binary Specification Loader could not be instantiated: nasdaq.nsmequities.totalview.itch.v1.0.pdf.xml. OmiSpecifications commit `fa3ed9cf` ("Update nasdaq sources") added seven new `Reference.xml` declarations for old Nasdaq TotalView ITCH versions (1.0, 2.0, 2.0.a, 3.0, 3.1, 3.1.f, 3.2). Each one's ASCII-price `<Source>` element declares `<Loader>Itch.NasdaqEquities.Ascii.Pdf</Loader>` (for example `OmiSpecifications/Nasdaq/NsmEquities/TotalView/Nasdaq.NsmEquities.TotalView.Itch.v1.0.Reference.xml:43`). No loader with that identifier is registered: `UniversalBinarySpecification/Assemblies/Omi.Binary.Specification.Loaders/Itch/Itch.NasdaqEquities.Pdf/Source/Source.cs:12` declares `public const string Identifier = "Itch.NasdaqEquities.Pdf"` (no `Ascii`), and the sibling declaration that already compiles, `Nasdaq.NsmEquities.TotalView.Itch.v4.0.Reference.xml:37`, uses exactly that name. `Scaled.Binary.Specification.Library.Loaders.TryGet` (`ScaledCompilers/Binary/Scaled.Binary.Specification.Builder/Library/Dictionaries/Loaders.cs:128-133`) looks the string up in a dictionary keyed by each loader's `Identifier` field and finds nothing, so `Load.Source.From` (`ScaledCompilers/Binary/Scaled.Binary.Specification.Builder/Source/Load.cs:20`) throws. Propose changing `<Loader>Itch.NasdaqEquities.Ascii.Pdf</Loader>` to `<Loader>Itch.NasdaqEquities.Pdf</Loader>` in all seven Reference.xml files; not applied. Covers: nasdaq.nsmequities.totalview.itch.v1.0, v2.0, v2.0.a, v3.0, v3.1, v3.1.f, v3.2.
