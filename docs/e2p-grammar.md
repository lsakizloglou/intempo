### E2P Grammar

```
Event          ::= STRING ':{' Action* '}'
Action         ::= Add | AddRef | Delete | DeleteRef | Modify
Add            ::= 'adds' ( ReferralByName ':' )? ID
                   ( '>>' Commit )? ( '[' AttributeAssignment* ']' )?
Delete         ::= 'deletes' ReferralByRetrieval
Modify         ::= 'modifies' Referral '[' AttributeAssignment* ']'
AddRef         ::= 'adds-ref' ID Referral '->' Referral
DeleteRef      ::= 'deletes-ref' ID Referral '->' Referral
Commit         ::= ID '(' Value ')'
Referral       ::= ReferralByName | ReferralByRetrieval
ReferralByRetrieval ::= '$' ID '(' Value ')'
ReferralByName ::= ID
AttributeAssignment ::= ID '=' Value
Value          ::= StringValue | ParameterValue
StringValue    ::= STRING
ParameterValue ::= '*p' INT ( ( '~*p' INT )? ( '+' INT )? ( '-' INT )? )
```