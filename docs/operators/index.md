# Expression Operator Overview

Fluxion mirrors MongoDB aggregation operators so you can compose complex transformations inside `$project`, `$addFields`, `$group`, and other stages. Browse the categories below, then use the sidebar for the complete alphabetical list.

## Comparison & Logic

- [$eq](eq.md), [$ne](ne.md), [$gt](gt.md), [$gte](gte.md), [$lt](lt.md), [$lte](lte.md)
- Set membership: [$in](in.md), [$nin](nin.md)
- Logical helpers: [$and](and.md), [$or](or.md), [$not](not.md), [$switch](switch.md), [$ifNull](ifNull.md)

## Math & Numeric

- Arithmetic: [$add](add.md), [$subtract](subtract.md), [$multiply](multiply.md), [$divide](divide.md), [$mod](mod.md)
- Rounding: [$ceil](ceil.md), [$floor](floor.md), [$round](round.md), [$trunc](trunc.md)
- Advanced math: [$pow](pow.md), [$sqrt](sqrt.md), [$exp](exp.md), [$ln](ln.md), [$log10](log10.md)

## Array Utilities

- Transformation: [$map](map.md), [$filter](filter.md), [$reduce](reduce.md)
- Structure: [$concatArrays](concatArrays.md), [$arrayElemAt](arrayElemAt.md), [$zip](zip.md)
- Analysis: [$size](size.md), [$slice](slice.md), [$reverseArray](reverseArray.md)

## String Processing

- Case & trimming: [$toLower](toLower.md), [$toUpper](toUpper.md), [$trim](trim.md), [$ltrim](ltrim.md), [$rtrim](rtrim.md)
- Substrings: [$substr](substr.md), [$substrBytes](substrBytes.md), [$substrCP](substrCP.md)
- Matching: [$split](split.md), [$strcasecmp](strcasecmp.md), [$regexMatch](regexMatch.md)

## Date & Time

- Conversion and extraction: [$toDate](toDate.md) (covers `$year`, `$month`, `$week`, etc. through options).
- Differences and arithmetic: [$add](add.md) with date operands, [$subtract](subtract.md), and [$convert](convert.md).

## Type & Object Utilities

- Conversion: [$convert](convert.md), [$toInt](toInt.md), [$toDouble](toDouble.md), [$toBool](toBool.md), [$toString](toString.md)
- Inspection: [$type](type.md), [$literal](literal.md)
- Object helpers: [$getField](getField.md), [$setField](setField.md), [$unsetField](unsetField.md), [$mergeObjects](mergeObjects.md), [$objectToArray](objectToArray.md), [$arrayToObject](arrayToObject.md)

## Special Execution

- [$function](function.md) for JavaScript-style custom logic with `body`, `args`, and optional `lang`.

Looking for something specific? Use the search box or the alphabetical list in the sidebar to jump straight to the operator you need.
