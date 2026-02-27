# ![](../pix/vrmllogo2.0.gif)

# The Virtual Reality Modeling Language

# Annex A. Grammar Definition

### (normative)

### ISO/IEC DIS 14772-1

#### 4 April 1997

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif) A.1 Table of contents and introduction

### A.1.1 Table of contents

This annex provides a detailed description of the grammar for each
syntactic element in VRML as described in the following subclauses. The
following table of contents lists the topics in this clause:

[A.1 Table of contents and introduction](#Introduction)

[A.1.1 \
Table of contents](#A.1.1)

[A.1.2 \
Introduction](#A.1.2)

[A.2 General](#General)

[A.3 Nodes](#Nodes)

[A.4 Fields](#Fields)

### A.1.2 Introduction

It is not possible to parse VRML files using a context-free grammar.
Semantic knowledge of the names and types of fields, eventIns, and
eventOuts for each node type (either built-in or user-defined using **PROTO**
or **EXTERNPROTO**) shall be used during parsing so that the parser
knows which field type is being parsed.

The '#' (0x23) character begins a comment wherever it appears outside
of quoted SFString or MFString fields. The '#' character and all
characters until the next line terminator comprise the comment and are
treated as whitespace.

The carriage return (0x0d), linefeed (0x0a), space (0x20), tab (0x09),
and comma (0x2c) characters are whitespace characters wherever they
appear outside of quoted SFString or MFString fields. Any number of
whitespace characters and comments may be used to separate the
syntactic entities of a VRML file. All reserved keywords are displayed
in boldface type.

Clause " [6\. Nodes Reference](nodesRef.html)"
contains a description of the allowed fields, eventIns and eventOuts
for all pre-defined node types. Some of the basic types that will
typically be handled by a lexical analyzer ( _sffloatValue_, _sftimeValue_, _sfint32Value_, and _sfstringValue_) have
not been formally specified. Clause " [5\. \
Fields and Events Reference](fieldsRef.html)" contains a more complete
description of their syntax.

The following conventions are used in the context-free grammar
specified in this clause:

- Terminal symbols, which appear literally in the VRML file, are
   specified in **bold**.

- Nonterminal symbols used in the grammar are specified in _italic_.

- Production rules begin with a nonterminal symbol and the sequence of
   characters "::=", and end with a semi-colon (";").

- Alternation for production rules is specified using the vertical-bar
   symbol ("\|").


![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif) A.2 General

_vrmlScene_::=
 _statements_ ;
 _statements_::=
 _statement_ \|
 _statement statements_ \|
 _empty_;
 _statement_::=
 _nodeStatement_ \|
 _protoStatement_ \|
 _routeStatement_ ;
 _nodeStatement_::=
 _node_ \|
 **DEF** _nodeNameId node_ \|
 **USE** _nodeNameId_ ;
 _protoStatement_::=
 _proto_ \|
 _externproto_ ;
 _protoStatements_ ::=
 _protoStatement_ \|
 _protoStatement protoStatements_ \|
 _empty_ ;
 _proto_::=
 **PROTO** _nodeTypeId_ **\[** _interfaceDeclarations_ **\] {** _protoBody_ **}** ;
 _protoBody_ ::=
 _protoStatements_ _node_ _statements_ ;
 _interfaceDeclarations_::=
 _interfaceDeclaration_ \|
 _interfaceDeclaration interfaceDeclarations_ \|
 _empty_ ;
 _restrictedInterfaceDeclaration_::=
 **eventIn** _fieldType_ _eventInId_ \|
 **eventOut** _fieldType eventOutId_ \|
 **field** _fieldType fieldId fieldValue_ ;
 _interfaceDeclaration_::=
 _restrictedInterfaceDeclaration_ \|
 **exposedField** _fieldType fieldId fieldValue_ ;
 _externproto_::=
 **EXTERNPROTO** _nodeTypeId_ **\[** _externInterfaceDeclarations_ **\]** _URLList_;
 _externInterfaceDeclarations_::=
 _externInterfaceDeclaration_ \|
 _externInterfaceDeclaration externInterfaceDeclarations_ \|
 _empty_ ;
 _externInterfaceDeclaration_::=
 **eventIn** _fieldType_ _eventInId_ \|
 **eventOut** _fieldType eventOutId_ \|
 **field** _fieldType fieldId_ \|
 **exposedField** _fieldType fieldId_ ;
 _routeStatement_::=
 **ROUTE** _nodeNameId_ **.** _eventOutId_ **TO** _nodeNameId_ **.** _eventInId_ ;
 _URLList_ ::=
 _mfstringValue_ ;
 _empty_ ::=

 ;

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif) A.3 Nodes

_node_::=
 _nodeTypeId_ **{** _nodeBody_ **}** \|
 **Script {** _scriptBody_ **}** ;
 _nodeBody_::=
 _nodeBodyElement_ \|
 _nodeBodyElement nodeBody_ \|
 _empty_ ;
 _scriptBody_::=
 _scriptBodyElement_ \|
 _scriptBodyElement scriptBody_ \|
 _empty_ ;
 _scriptBodyElement_::=
 _nodeBodyElement_ \|
 _restrictedInterfaceDeclaration_ \|
 **eventIn** _fieldType_ _eventInId_ **IS** _eventInId_ \|
 **eventOut** _fieldType eventOutId_ **IS** _eventOutId_
 \|
 **field** _fieldType fieldId_ **IS** _fieldId_ ;
 _nodeBodyElement_::=
 _fieldId fieldValue_ \|
 _fieldId_ **IS** _fieldId_ \|
 _eventInId_ **IS** _eventInId_ \|
 _eventOutId_ **IS** _eventOutId_ \|
 _routeStatement_ \|
 _protoStatement_ ;
 _nodeNameId_::=
 _Id_ ;
 _nodeTypeId_::=
 _Id_ ;
 _fieldId_::=
 _Id_ ;
 _eventInId_::=
 _Id_ ;
 _eventOutId_::=
 _Id_ ;
 _Id_::=
 _IdFirstChar_ \|
 _IdFirstChar IdRestChars_ ;
 _IdFirstChar_::=

 Any ISO-10646 character encoded using UTF-8 except: 0x30-0x39,
 0x0-0x20, 0x22, 0x23, 0x27, 0x2b, 0x2c, 0x2d, 0x2e, 0x5b, 0x5c, 0x5d,
 0x7b, 0x7d, 0x7f ;
 _IdRestChars_::=

 Any number of ISO-10646 characters except: 0x0-0x20, 0x22, 0x23, 0x27,
 0x2c, 0x2e, 0x5b, 0x5c, 0x5d, 0x7b, 0x7d, 0x7f ;

![](../pix/vrmlbar.gif)

## ![](../../pix/cube.gif) A.4 Fields

_fieldType_::=
 **MFColor** \|
 **MFFloat** \|
 **MFInt32** \|
 **MFNode** \|
 **MFRotation** \|
 **MFString** \|
 **MFTime** \|
 **MFVec2f** \|
 **MFVec3f** \|
 **SFBool** \|
 **SFColor** \|
 **SFFloat** \|
 **SFImage** \|
 **SFInt32** \|
 **SFNode** \|
 **SFRotation** \|
 **SFString** \|
 **SFTime** \|
 **SFVec2f** \|
 **SFVec3f** ;
 _fieldValue_::=
 _sfboolValue_ \|
 _sfcolorValue_ \|
 _sffloatValue_ \|
 _sfimageValue_ \|
 _sfint32Value_ \|
 _sfnodeValue_ \|
 _sfrotationValue_ \|
 _sfstringValue_ \|
 _sftimeValue_ \|
 _sfvec2fValue_ \|
 _sfvec3fValue_ \|
 _mfcolorValue_ \|
 _mffloatValue_ \|
 _mfint32Value_ \|
 _mfnodeValue_ \|
 _mfrotationValue_ \|
 _mfstringValue_ \|
 _mftimeValue_ \|
 _mfvec2fValue_ \|
 _mfvec3fValue_ ;
 _sfboolValue_::=
 **TRUE** \|
 **FALSE** ;
 _sfcolorValue_::=
 _float float float_ ;
 _sffloatValue_::=

 ... floating point number in ANSI C floating point format...
 _sfimageValue_::=
 _int32 int32 int32 int32 ...__sfint32Value_::=

 \[\[ **+**\]\| **-**\]{\[ **0-9**\]+\| **0x**\[ **0-9a-fA-F**\] **+**};
 _sfnodeValue_::=
 _nodeStatement_ \|
 **NULL** ;
 _sfrotationValue_::=
 _float float float float_ ;
 _sfstringValue_::=
 **" _.\*_"** ... double-quotes must be \\",
 backslashes must be \\\...
 _sftimeValue_::=

 ... double-precision number in ANSI C floating point format...
 _mftimeValue_::=
 _sftimeValue_ \|
 **\[ \]** \|
 **\[** _sftimeValues_ **\]** ;
 _sftimeValues_::=
 _sftimeValue_ \|
 _sftimeValue sftimeValues_ ;
 _sfvec2fValue_::=
 _float float_ ;
 _sfvec3fValue_::=
 _float float float_ ;
 _mfcolorValue_::=
 _sfcolorValue_ \|
 **\[ \]** \|
 **\[** _sfcolorValues_ **\]** ;
 _sfcolorValues_::=
 _sfcolorValue_ \|
 _sfcolorValue sfcolorValues_ ;
 _mffloatValue_::=
 _sffloatValue_ \|
 **\[ \]** \|
 **\[** _sffloatValues_ **\]** ;
 _sffloatValues_::=
 _sffloatValue_ \|
 _sffloatValue_ _sffloatValues_ ;
 _mfint32Value_::=
 _sfint32Value_ \|
 **\[ \]** \|
 **\[** _sfint32Values_ **\]** ;
 _sfint32Values_::=
 _sfint32Value_ \|
 _sfint32Value sfint32Values_ ;
 _mfnodeValue_::=
 _nodeStatement_ \|
 **\[ \]** \|
 **\[** _nodeStatements_ **\]** ;
 _nodeStatements_::=
 _nodeStatement_ \|
 _nodeStatement nodeStatements_ ;
 _mfrotationValue_::=
 _sfrotationValue_ \|
 **\[ \]** \|
 **\[** _sfrotationValues_ **\]** ;
 _sfrotationValues_::=
 _sfrotationValue_ \|
 _sfrotationValue sfrotationValues_ ;
 _mfstringValue_::=
 _sfstringValue_ \|
 **\[ \]** \|
 **\[** _sfstringValues_ **\]** ;
 _sfstringValues_::=
 _sfstringValue_ \|
 _sfstringValue sfstringValues_ ;
 _mfvec2fValue_::=
 _sfvec2fValue_ \|
 **\[ \]** \|
 **\[** _sfvec2fValues_ **\]** ;
 _sfvec2fValues_::=
 _sfvec2fValue_ \|
 _sfvec2fValue sfvec2fValues_ ;
 _mfvec3fValue_::=
 _sfvec3fValue_ \|
 **\[ \]** \|
 **\[** _sfvec3fValues_ **\]** ;
 _sfvec3fValues_::=
 _sfvec3fValue_ \|
 _sfvec3fValue_ _sfvec3fValues_ ;

![](../pix/vrmlbar.gif)

```
https://graphics.stanford.edu/courses/cs248-98-fall/Assignments/Assignment3/VRML2_Specification/spec/part1/grammar.html

```

