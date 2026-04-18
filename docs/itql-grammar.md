### ITQL Grammar

```
Condition      ::= And ( 'declarations' '{' Declaration* '}' )?
And            ::= Until ( 'AND' Until )?
Until          ::= Since ( 'U' INTERVAL Since )?
Since          ::= BasicCondition ( 'S' INTERVAL BasicCondition )?
BasicCondition ::= 'true' | Not | Exists | '(' And ')' | Proxy
Not            ::= '!' BasicCondition
Exists         ::= 'E' ( ( StoryPattern | Proxy ) Binding?
                       | '(' ( StoryPattern | Proxy ) Binding? ( ',' And )? ')' )
StoryPattern   ::= ID '{' ( StoryPatternObject | StoryPatternLink
                           | '[' StringExpression ']' )* '}'
StoryPatternObject ::= ID ':' ScopedType
StoryPatternLink   ::= QualifiedName '-' ScopedFeature '->' QualifiedName
StringExpression   ::= ID ':' STRING
ScopedType     ::= ID ( '::' ID )?
ScopedFeature  ::= ID
Proxy          ::= '$' QualifiedName
Binding        ::= '[' Mapping* ']'
Mapping        ::= ID '->' ID
Declaration    ::= NamedExpression | StoryPattern
NamedExpression ::= ID ':' And
QualifiedName  ::= ID ( '.' ID )*
INTERVAL       ::= '[' DIGITS UNIT? ',' ( DIGITS UNIT? | '*' ) ']' ( 'oc' | 'co' | 'oo' )?
```

Where `UNIT` is one of `d` (days), `m` (months), `y` (years). If omitted, timestamps are interpreted in the unit of the log (typically seconds or milliseconds). The suffix `oc`, `co`, `oo` controls open/closed interval boundaries (default is closed-closed).