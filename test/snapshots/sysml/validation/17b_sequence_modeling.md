# META
~~~ini
description=SysML Validation (17-Sequence Modeling): 17b-Sequence-Modeling
type=file
~~~
# SOURCE
~~~sysml
package '17b-Sequence-Modeling' {
	private import ScalarValues::*;
	private import PayloadDefinitions::*;

	package PayloadDefinitions {
	    item def Subscribe {
	    	attribute topic : String;
	    	ref part subscriber;
	    }
	    
		item def Publish {
			attribute topic : String;
			ref publication;
		}
		
		item def Deliver {
			ref publication;
		}
	}

	occurrence def PubSubSequence {
		part producer[1] {
			event publish_message.sourceEvent;
		}
		
		message publish_message of Publish[1];
		
		part server[1] {
			event subscribe_message.targetEvent;
			then event publish_message.targetEvent;
			then event deliver_message.sourceEvent;
		}
		
		message subscribe_message of Subscribe[1];
		message deliver_message of Deliver[1];
		
		part consumer[1] {
			event subscribe_message.sourceEvent;
			then event deliver_message.targetEvent;
		}
	}
}
~~~
# DIAGNOSTICS
~~~sexpr
(fixture-diagnostics
  (document "memory://snapshot/17b_sequence_modeling.md"
    (diagnostics
      (diagnostic
        (severity warning)
        (code "unresolved_import_target")
        (source "semantic")
        (range (start 1 16) (end 1 31))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_type_reference")
        (source "semantic")
        (range (start 6 24) (end 6 30))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_type_reference")
        (source "semantic")
        (range (start 11 21) (end 11 27))
      )
    )
  )
)
~~~
# SMG
~~~sexpr
(semantic-model
  (publication (phase resolved) (completeness complete) (has-evaluation false) (source-digest "blake3:0c687464fe1ae65accabbc86a19fb6159fbe2c0aa57e95ac0ef3538be111c940") (contract-version "parser-owned-resolution-v1"))
  (declarations
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling"))) (kind package) (membership (kind owning) (visibility default)))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (anonymous (kind import) (ordinal 0))))) (kind import) (membership (kind import) (visibility private)) (authored (membership (kind import) (visibility private)) (relationships (namespaceImport (reference "ScalarValues") (import (shape namespace) (recursive false)))))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (anonymous (kind import) (ordinal 1))))) (kind import) (membership (kind import) (visibility private)) (authored (membership (kind import) (visibility private)) (relationships (namespaceImport (reference "PayloadDefinitions") (import (shape namespace) (recursive false)))))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions"))) (kind package) (membership (kind owning) (visibility default)))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions::Deliver"))) (kind item-def) (membership (kind owning) (visibility default)))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions::Deliver::publication"))) (kind ref) (membership (kind feature) (visibility default)))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions::Publish"))) (kind item-def) (membership (kind owning) (visibility default)))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions::Publish::publication"))) (kind ref) (membership (kind feature) (visibility default)))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions::Publish::topic"))) (kind attribute) (membership (kind feature) (visibility default)) (authored (membership (kind feature) (visibility default)) (relationships (featureTyping (reference "String"))))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions::Subscribe"))) (kind item-def) (membership (kind owning) (visibility default)))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions::Subscribe::subscriber"))) (kind ref) (membership (kind feature) (visibility default)))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions::Subscribe::topic"))) (kind attribute) (membership (kind feature) (visibility default)) (authored (membership (kind feature) (visibility default)) (relationships (featureTyping (reference "String"))))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PubSubSequence"))) (kind occurrence-def) (membership (kind owning) (visibility default)))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PubSubSequence::consumer"))) (kind part) (membership (kind feature) (visibility default)))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (anonymous (kind occurrence) (ordinal 0))))) (kind occurrence) (membership (kind feature) (visibility default)))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (anonymous (kind occurrence) (ordinal 1))))) (kind occurrence) (membership (kind feature) (visibility default)))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PubSubSequence::deliver_message"))) (kind flow) (membership (kind feature) (visibility default)) (authored (membership (kind feature) (visibility default)) (relationships (flowPayloadType (reference "Deliver"))))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PubSubSequence::producer"))) (kind part) (membership (kind feature) (visibility default)))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (anonymous (kind occurrence) (ordinal 0))))) (kind occurrence) (membership (kind feature) (visibility default)))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PubSubSequence::publish_message"))) (kind flow) (membership (kind feature) (visibility default)) (authored (membership (kind feature) (visibility default)) (relationships (flowPayloadType (reference "Publish"))))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PubSubSequence::server"))) (kind part) (membership (kind feature) (visibility default)))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (anonymous (kind occurrence) (ordinal 0))))) (kind occurrence) (membership (kind feature) (visibility default)))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (anonymous (kind occurrence) (ordinal 1))))) (kind occurrence) (membership (kind feature) (visibility default)))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (anonymous (kind occurrence) (ordinal 2))))) (kind occurrence) (membership (kind feature) (visibility default)))
    (declaration (id (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PubSubSequence::subscribe_message"))) (kind flow) (membership (kind feature) (visibility default)) (authored (membership (kind feature) (visibility default)) (relationships (flowPayloadType (reference "Subscribe"))))
  )
  (references
    (reference (id (source (node (document "memory://snapshot/17b_sequence_modeling.md") (anonymous (kind import) (ordinal 0))))) (kind namespaceImport) (ordinal 0))
      (authored-target "ScalarValues")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/17b_sequence_modeling.md") (anonymous (kind import) (ordinal 1))))) (kind namespaceImport) (ordinal 0))
      (authored-target "PayloadDefinitions")
      (outcome (status resolved) (target (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions")))))
    (reference (id (source (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions::Publish::topic"))) (kind featureTyping) (ordinal 0))
      (authored-target "String")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions::Subscribe::topic"))) (kind featureTyping) (ordinal 0))
      (authored-target "String")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PubSubSequence::deliver_message"))) (kind flowPayloadType) (ordinal 0))
      (authored-target "Deliver")
      (outcome (status resolved) (target (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions::Deliver")))))
    (reference (id (source (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PubSubSequence::publish_message"))) (kind flowPayloadType) (ordinal 0))
      (authored-target "Publish")
      (outcome (status resolved) (target (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions::Publish")))))
    (reference (id (source (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PubSubSequence::subscribe_message"))) (kind flowPayloadType) (ordinal 0))
      (authored-target "Subscribe")
      (outcome (status resolved) (target (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions::Subscribe")))))
  )
  (relationships
    (relationship (kind flowPayloadType) (source (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PubSubSequence::deliver_message"))) (target (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions::Deliver"))) (provenance authored) (authored-reference (source (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PubSubSequence::deliver_message"))) (kind flowPayloadType) (ordinal 0)))
    (relationship (kind flowPayloadType) (source (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PubSubSequence::publish_message"))) (target (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions::Publish"))) (provenance authored) (authored-reference (source (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PubSubSequence::publish_message"))) (kind flowPayloadType) (ordinal 0)))
    (relationship (kind flowPayloadType) (source (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PubSubSequence::subscribe_message"))) (target (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions::Subscribe"))) (provenance authored) (authored-reference (source (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PubSubSequence::subscribe_message"))) (kind flowPayloadType) (ordinal 0)))
  )
  (evaluation
  )
)
~~~
# NAVIGATION
~~~sexpr
(navigation
  (query (document "memory://snapshot/17b_sequence_modeling.md") (range (start 1 16) (end 1 31)) (probe (position 1 16))
    (reference (id (source (node (document "memory://snapshot/17b_sequence_modeling.md") (anonymous (kind import) (ordinal 0))))) (kind namespaceImport) (ordinal 0) (authored-target "ScalarValues")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/17b_sequence_modeling.md") (range (start 2 16) (end 2 37)) (probe (position 2 16))
    (reference (id (source (node (document "memory://snapshot/17b_sequence_modeling.md") (anonymous (kind import) (ordinal 1))))) (kind namespaceImport) (ordinal 0) (authored-target "PayloadDefinitions")
      (outcome (status resolved) (target (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions")))))
  )
  (query (document "memory://snapshot/17b_sequence_modeling.md") (range (start 11 21) (end 11 27)) (probe (position 11 21))
    (reference (id (source (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions::Publish::topic"))) (kind featureTyping) (ordinal 0) (authored-target "String")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/17b_sequence_modeling.md") (range (start 6 24) (end 6 30)) (probe (position 6 24))
    (reference (id (source (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions::Subscribe::topic"))) (kind featureTyping) (ordinal 0) (authored-target "String")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/17b_sequence_modeling.md") (range (start 34 29) (end 34 36)) (probe (position 34 29))
    (reference (id (source (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PubSubSequence::deliver_message"))) (kind flowPayloadType) (ordinal 0) (authored-target "Deliver")
      (outcome (status resolved) (target (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions::Deliver")))))
  )
  (query (document "memory://snapshot/17b_sequence_modeling.md") (range (start 25 29) (end 25 36)) (probe (position 25 29))
    (reference (id (source (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PubSubSequence::publish_message"))) (kind flowPayloadType) (ordinal 0) (authored-target "Publish")
      (outcome (status resolved) (target (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions::Publish")))))
  )
  (query (document "memory://snapshot/17b_sequence_modeling.md") (range (start 33 31) (end 33 40)) (probe (position 33 31))
    (reference (id (source (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PubSubSequence::subscribe_message"))) (kind flowPayloadType) (ordinal 0) (authored-target "Subscribe")
      (outcome (status resolved) (target (node (document "memory://snapshot/17b_sequence_modeling.md") (qualified-name "17b-Sequence-Modeling::PayloadDefinitions::Subscribe")))))
  )
)
~~~
