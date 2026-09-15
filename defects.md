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

fixed-length ASCII element has no recognized Filled trait (FileForText throws) (24 models)

first error: FileForText: fixed-length ASCII element 'Protocol Version' (address: 'packet.packetheader.session.protocolversion', bytes: 3) has no recognized Filled trait (expected Zeros, NullCharacters, or Spaces). Fix the model/compiler to add the correct Filled trait for this field.

## utf16-translation

Ascii element carries a Translation=Utf16 trait the generator does not classify (16 models)

first error: Specified argument was out of the range of valid values. (Parameter 'Unknown Type: [Field] Header')

## floating-point-field

IEEE-754 double (Translation=FloatingPoint) field the generator does not classify (57 models)

first error: Specified argument was out of the range of valid values. (Parameter 'Unknown Type: [Field] Current Funding')

## big-endian-enum-overlay

multi-byte big-endian integer enum cannot be exposed by direct memory overlay (35 models)

first error: Integer enum 'Deal Source' (address: 'serverpacket.serversoupbintcppacket.serverpayload.sequenceddatapacket.sequencedmessage.orderexecutedmessage.dealsource') is 2 bytes and big-endian. ZeroCopy cannot expose a multi-byte big-endian enum directly from a memory overlay. Use a supported little-endian representation or add an endian-aware wrapper type.

## sbe-block-length-ambiguous

SBE message-header block-length field could not be uniquely identified (1 models)

first error: Model 'SmallX.OrderBookFeed.Sbe.v2.2': message-header block-length field could not be uniquely identified in composite 'Message Header' (packet.optiqmessage.messageheader). The original message targets do not state one shared Size dependency for the root block length.

## accessor-missing-size-trait

fixed-layout accessor field is missing required Size trait (19 models)

first error: Model 'A2X.A2XEquities.UdpHeader.Amd.v1': fixed-layout accessor field is missing required Size trait. This is a model/compiler defect. Field='Payload' (Field, packet.message.payload, 0 bytes).

## cs0031-sbyte-overflow

generates clean, fails build: CS0031 sbyte-constant overflow in the Types emitter (9 models)

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

generates clean, fails build: CS1061 'LpRoleOptional' does not contain a definition for 'Value' (40 models)

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

element declares more than one Value fact with different data; generator refuses by design (8 models)

first error: Element 'Secondary Exec Id' (packet.simpleopenframe.payload.executionreporttrademessage.secondaryexecid) declares more than one 'FixTag' value with different data ('8' and '527'). The generator emits one 'FixTag' constant per element.

## bitfield-no-endian

NEW:bitfield-no-endian (16 models)

first error: Bitfield container 'Exec Inst' (address: 'packet.data.unsequencedmessage.sbemessage.payload.newordersinglemessage.execinst') is 2 bytes but has no explicit Endian trait. Add exactly one Endian trait with value Big or Little upstream.

## multi-packet-header

NEW:multi-packet-header (RESOLVED 2026-09-13)

The PacketHeader.Find fix removed this first error. Six models now pass end to
end. The other 24 models moved to their actual first-error buckets above.

## enum-multibit-bitfield

enum-valued multi-bit bitfield children not implemented (10 models)

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

leaf field with no Translation trait, or no Endian trait on a multi-byte Integer (3 models)

first error: Specified argument was out of the range of valid values. (Parameter 'Unknown integer type: [Field] Strike Price Mantissa')

## nullable-no-value-entry

Nullable trait with no Nullable value entry (11 models)

first error: Element 'Execution Mode' has the Nullable trait but no Nullable value entry — expected a Values entry with type=Nullable to carry the sentinel data.

## unresolved-element-argument

unresolved 'element' ArgumentOutOfRangeException (9 models)

first error: Specified argument was out of the range of valid values. (Parameter 'element')

## unresolved-value-argument

unresolved 'value' ArgumentOutOfRangeException (2 models)

first error: Specified argument was out of the range of valid values. (Parameter 'value')

## bitfield-nonprimitive-width

bitfield container spans a non-whole-primitive width (2 models)

first error: Bitfield container 'Flags' (address: 'packet.messages.message.payload.trademessage.flags') spans 57 bits, which is not a whole-primitive width (expected 8/16/32/64). Non-byte-aligned or over-wide bitfield containers have no C# primitive backing and would truncate child masks; fix the model/compiler upstream or add a wide-backed bitfield template.

## nonstandard-decimal-width

NEW:nonstandard-decimal-width (4 models)

first error: Element '[Field] Stop Px' is a non-standard-width (99-byte) decimal field. NonStandard does not apply Factor scaling; fix the model/compiler upstream.

## duplicate-struct-name

NEW:duplicate-struct-name (4 models)

first error: Model 'Jpx.FseEquities.MarketByOrder.Flex.v1.1': two emitted structs collide on C# declaration name 'PacketHeader' — 'udppacket.packetheader' and 'tcppacket.packetheader'. Message-level qualification was insufficient.

## char-enum-one-char

NEW:char-enum-one-char (3 models)

first error: Character enum 'Quote Condition' (packet.message.payload.consolidatedsinglesidedquotemessage.quotecondition) value 'Empty Quote' has data 'OxOO'. A one-byte ASCII enum value must contain exactly one character.

## single-branch-case

single unconditional Branch case dispatch not supported by ZeroCopy (1 models)

first error: Cannot emit ZeroCopy library artifacts for 'Cboe.GapDetection.Pitch.v3': the dispatch element carries a single unconditional Branch case with no Dependency parameter, so there is no discriminator field to key on. ZeroCopy does not support single-unconditional-case dispatch.

## length-disambiguated-dispatch

NEW:length-disambiguated-dispatch (1 models)

first error: Model 'Cboe.TitaniumConsolidated.OneEquitiesTcp.Pitch.v1.4.13': dispatch element 'packet.messages.message.payload' is classified as LengthDisambiguatedDispatch. ZeroCopy does not support shared wire codes.

## mixed-size-rule-dispatch-targets

NEW:mixed-size-rule-dispatch-targets (1 models)

first error: Model 'Siac.Opra.Output.Obi.v6.3.Hft': some dispatch targets of 'packet.message.payload' carry a Size rule and some do not.

## remaining-payload-no-size-rule

NEW:remaining-payload-no-size-rule (RESOLVED 2026-09-14)

The declared-message-characteristic slice's dispatch gate removal let
Imperative.IntelligentCross.DepthOfBook.Aspen.v1.11 reach and pass generation
and build.

## cs0103

CS0103 (1 models)

first error: error CS0103: The name '...

## cs0155

CS0155 (6 models)

first error: error CS0155: The type caught or thrown must be derived f...

## cs0246

CS0246 (3 models)

first error: error CS0246: The type or namespace name '...

## cs0266-ulong-to-long

CS0266 (47 models)

first error: error CS0266: Cannot implicitly convert type 'ulong' to 'long'. An explicit conv...

## cs0542-value-member-name

CS0542 (6 models)

first error: error CS0542: 'Value': member na...

## cs0029

CS0029 (5 models)

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

# Classes

## §15

UNSUPPORTED - multi-transport (multi-tree) models (0 models.)


## §16

nullable/default sentinel unsupported (0 models.)


## §18

one-byte length field (0 models.)


## §25

untriaged Classes-only InvalidOperationException (Siac.Opra.Recipient.Obi.v4.0) (0 models.)


## §27

Block Header loses its Header characteristic to a duplicate-identifier struct/type (0 models.)


## §28

B3.B3Derivatives.BinaryEntryPoint.Sbe.v8.4 CS0103 (0 models.)


## §29

MessageTypeConstant '0x..' was expected to be a hex literal (0 models.)


## §30

IntegerType.For: wire width is not supported (0 models. 2026-09-09: both members)

(Cboe.BxeEquities/CxeEquities.AuctionFeed.AsciiPitch.v1.4) now fail generation with
'Field 'Price' is a decimal field with unsupported width 19', the same message
template as §41 (decimal field unsupported width). Merged into §41.

## §33

CS0103 the name 'R' does not exist in the current context (0 models. 2026-09-09:)

both members (Miax.PearlEquities.DepthOfMarket.Mach.v1.3.d,
Miax.PearlEquities.TopOfMarket.Mach.v1.1.c) now pass end to end; see the dropped list.

## §35

filed 2026-09-01, never denylisted (0 models.)


## §36

MemberFieldHex - fixed 2026-09-02 (0 models.)


## §37

OverflowException: value too large or too small (0 models.)


## §39

MessageTypeConstant '<XX>' is not a valid literal (0 models.)


## §42

ArgumentNullException: Value cannot be null (0 models.)


## §47

dispatch element with an unsupported Branch shape (0 models.)


## §2B

String field has no recognized Filled trait (0 models. 2026-09-12: both)

members (Omi.Sbe.Example.Sbe.v1, Omi.Sbe.Example.Sbe.v2) now pass end to end
(verify-classes-all unexpected-generation-success after the Classes 4 conversion);
confirmed with sj check-classes and sj diff-generated classes vs 3ec23c45 (only the
added Dispatch.cs 'using System.Buffers.Binary;' line differs). Ledger was stale.

## §61

multi-byte integer field has no Endian trait (0 models. This section was)

mislabeled §59 in the live block (a pre-existing collision with the unrelated,
still-active §59 FloatingPoint section below; renumbered here for clarity, not
re-triaged). 2026-09-12: both members (Cboe.C1Options.MarketDataFeed.Csm.v1.4.2,
Cboe.C1Options.OpeningAuction.Csm.v1.0) now pass end to end (verify-classes-all
unexpected-generation-success after the Classes 4 conversion); confirmed with sj
check-classes and sj diff-generated classes vs 3ec23c45 (byte-identical). Ledger
was stale.

## §54

group field offset does not match the enclosing message layout (0 models.)

2026-09-09: all 9 former members now fail earlier in generation with a different
error. The 7 B3.B3Derivatives.BinaryEntryPoint.Sbe.v7.0/v7.1/v8.0/v8.1/v8.2/v8.3/v8.4
models now raise 'Count field
...crosssidesgroups.groupsizeencoding.numingroup' is outside the parsed property
path...' (the §34 message template); moved to §34. The 2 Iex.IexOptions.MarketData.
Sbe.v1.03 / Iex.IexOptions.Session.Sbe.v1.0 models now raise 'Branch target ... is
neither a generated message nor a branch container with a direct dispatch child'
(the §53 message template); moved to §53.

## §20

duplicate-address sibling elements rejected by generator invariant (0 models.)

2026-09-13: all 14 members (Cboe.ByxEquities/BzxEquities/BzxOptions/C1Options/
C2Options/EdgaEquities/EdgxEquities/EdgxOptions.MulticastDepthOfBook.
Pitch/Spin.v2.41.66) now fail generation earlier ('Unsupported length field byte
width: 1. Expected 2 or 4.', the §38 template) against the freshly compiled
1083-model corpus; moved to §38.

## §26

CS0102 duplicate FutureLegList/ParseStarted/FailedPrefixField emission (0 models.)

2026-09-13: both members (Cboe.CfeFutures.MulticastDepthOfBook.Pitch.v1.1.12,
v1.1.6) now fail generation earlier ('Unsupported length field byte width: 1.
Expected 2 or 4.', the §38 template) against the freshly compiled 1083-model
corpus; moved to §38.

## §34

count field is outside the parsed property path (0 models.)

2026-09-13: the sole member (Cboe.C1Options.MarketLevel2.Csm.v1.0.4) now fails
generation earlier ('Branch target ... is neither a generated message nor a
branch container with a direct dispatch child', the §53 template) against the
freshly compiled 1083-model corpus; moved to §53.

## §48

CS0108 AdapFlagsField.Clear hides the inherited member (0 models.)

2026-09-13: all 4 members (Cboe.ByxEquities/BzxEquities/EdgaEquities/
EdgxEquities.SummaryDepth.Pitch.v1.0.4) now fail generation earlier ('Unsupported
length field byte width: 1. Expected 2 or 4.', the §38 template) against the
freshly compiled 1083-model corpus; moved to §38.

## §51

FormatException: input string was not in a correct format (0 models.)

2026-09-13: the sole member (Cboe.DxeDerivatives.MulticastDepthOfBook.Pitch.
v1.11) now fails generation earlier ('Unsupported length field byte width: 1.
Expected 2 or 4.', the §38 template) against the freshly compiled 1083-model
corpus; moved to §38.

## §60

leaf field with no Translation trait and no Endian trait; Field.Kind has nothing to dispatch on (0 models.)

2026-09-13: all 7 members now pass end to end (verify-classes-all
unexpected-generation-success against the freshly compiled 1083-model corpus).
Ledger was stale.

## §63

group field does not fit before the message dispatch field in a size-prefixed frame (0 models.)

2026-09-13: all 7 B3.B3Derivatives.BinaryEntryPoint.Sbe.v7.0/v7.1/v8.0/v8.1/
v8.2/v8.3/v8.4 members now pass end to end (verify-classes-all
unexpected-generation-success against the freshly compiled 1083-model corpus).
Ledger was stale.

## §52

String field has no recognized Justified trait (70 models.)

first error: String field 'ProtocolVersion' has no recognized Justified trait. Fix the model/compiler upstream.

## §43

Ascii element carries a Translation=Utf16 trait the generator does not classify (91 models.)

first error: Field.Kind: cannot classify element. The field is UTF-16-encoded text (Translation=Utf16); Field.Kind has no Utf16 branch. This is the

## §59

IEEE-754 double (Translation=FloatingPoint) field the generator does not classify (61 models.)

first error: Field.Kind: cannot classify element. Address='packet.message.payload.newordermessage.orderid', Name='Order Id', ByteWidth=8,

## §32

SentinelLiteral.For: archetype is not supported (91 models.)

first error: SentinelLiteral.For: archetype 'Bitfield' is not supported for field 'MatchEventIndicatorMatchEventIndicatorOptional'.

## §24

positive decimal exponent unsupported (14 models.)

first error: Element 'Trading Value' has a positive decimal exponent (3). Positive exponents require multiply-on-parse semantics; update the generator templates before addin...

## §38

size dependency field width unsupported (126 models.)

first error: Unsupported length field byte width: 1. Expected 2 or 4.
2026-09-13: reworded again (was 'Unsupported wire primitive byte width: 1. Expected
  2, 4, or 8.'); same underlying guard, wording has flipped between these two forms
  across passes. Also broadened sharply: grew from 9 to 126 members, absorbing all of
  former §20 (14), §26 (2), §48 (4), and §51 (1), plus 98 models that generated
  cleanly before this pass. See the top-of-file 2026-09-13 note.

## §41

decimal field with unsupported width (18 models.)

first error: Field 'PricePrice10' is a decimal field with unsupported width 10.

## §53

Branch target is neither a generated message nor a branch container (108 models.)

first error: Model 'B3.B3Derivatives.BinaryUmdf.Sbe.v1.6': Branch target 'Sequence Reset Message' (packet.message.payload.sequenceresetmessage) is neither a generated messag...

## §55

counted element has a non-integer count field (5 models.)

first error: Count field 'packet.messagebody.complexorderinstrumentkeysmessage.numberoflegs' must use a standard integer width.

## §56

message count dependency field is not inside the packet header (2 models.)

first error: Model 'A2X.A2XEquities.Rtmdf.Amd.v1.3.2': message count dependency field 'Message Count' (packet.messagecount) is not inside packet header 'Message Header' (packet.message.messageheader).
2026-09-13: all 6 former Siac Cqs/Cts members now pass end to end
  (unexpected-generation-success); the 1 remaining former member, Siac.Opra.
  Output.Obi.v4.0, now fails earlier ('Branch rule on element ... has 15
  Dependency parameters; at most one is valid', the §31 template) and moved to §31.
  2 new A2X models take its place on this same message-count-header shape.

## §45

enum-valued multi-bit bitfield children not implemented (9 models.)

first error: Bitfield child 'Party Role' (address: 'packet.message.payload.ioiaddmessage.tableselect1.partyrole') carries enum values, but enum-valued multi-bit bitfield chi...

## §44

message has no matching branch case in its dispatch element (4 models.)

first error: Message 'Delta Update Message' (packet.payload.deltaupdatemessages.deltaupdatemessage) has no matching branch case in dispatch element 'packet.payload'. Every d...

## §31

branch rule carries an unexpected number of Dependency parameters (5 models.)

first error: Branch rule on element 'Equity And Index Last Sale Message Payload' (packet.message.payload.equityandindexlastsalecategory.equityandindexlastsalemessagepayload)...

## §46

composite-key dispatch target has no body (2 models.)

first error: Model 'Siac.Cqs.Output.Cta.v2.10.Hft': composite-key dispatch target 'packet.message.messagepayload.startofdaymessage' has no body.

## §50

Bitfield.BackingType: unsupported byte width (2 models.)

first error: Field 'Flags' is a bitfield with unsupported width 7.

## §62

field has no generated shared field class for its wire digest (1 models.)

first error: Field 'pillarstreammessage.seqmsg.sequencedmessage.newordermessage.optionalorderaddon.

## §40

unresolved ArgumentOutOfRangeException (9 models.)

first error: Specified argument was out of the range of valid values. (Parameter 'element')

## §64

frame-header size dependency field is not 2 or 4 bytes wide (2 models.)

first error: Model 'Aquis.AquisEquities.Replay.Amd.v4.0': size dependency field 'Msg Length' (1 bytes) must be 2 or 4 bytes wide.
2026-09-13: the former sole member, SmallX.OrderBookFeed.Sbe.v2.2, now passes end to
  end (unexpected-generation-success); 2 new Aquis models take its place on this same
  frame-header shape.

## §65

dispatch collision targets share one payload length (1 models.)

first error: Model 'Cboe.TitaniumConsolidated.OneEquitiesTcp.Pitch.v1.4.13': dispatch collision targets share one payload length.

## §66

branch element has Count, Payload(Buffer=Rest), and Branch rules but no Size rule (1 models.)

first error: Model 'Imperative.IntelligentCross.DepthOfBook.Aspen.v1.11': element 'Message' (packet.message) has Count, Payload(Buffer=Rest), and runtime Branch rules but no Size rule/dependency. ZeroCopy length-prefixed packet reader generation requ...

## §67

Size rule carries more than one Operator parameter (1 models.)

first error: Size rule has 2 Operator parameters; at most one is valid for this operation.

## §57

CS0101 the namespace already contains a definition for the type (6 models.)

first error: error CS0101: The namespace 'Cboe.C1Options.BinaryOrderEntry.Boe3' already contains a definition for 'LegPositionEffect'

## §58

CS0104 'Exception' is an ambiguous reference (6 models.)

first error: error CS0104: 'Exception' is an ambiguous reference between 'Hkex.HkexSecurities.Index.Omd.Exception' and 'System.Exception'

## §68

CS0102 duplicate 'PartiesGroupList' emission (2 models.)

first error: error CS0102: The type 'SequencedMessage' already contains a definition for 'PartiesGroupList'

## §69

CS0103 'MessageHeader' does not exist in the RoundTrip harness (3 models.)

first error: error CS0103: The name 'MessageHeader' does not exist in the current context
