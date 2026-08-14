# META
~~~ini
description=SysML Example (Arrowhead Framework): AHFSequences
type=file
~~~
# SOURCE
~~~sysml
// ** This is the Norwegian use-case for Arrowhead Framework */
package AHFNorwaySequences {
	// Here we show sequences of the Norwegian use-case
	private import AHFProfileLib::*;
	private import AHFCoreLib::*;
	private import AHFNorway::*;
	private import ScalarValues::*;
	
	part AHFN_LocalCloudDD_Seqs = AHFNorway_LocalCloudDD{
		occurrence def APIS_transfer_lifetime {			
			// lifetime orderings 
			ref part tlc = AHFNorway_LocalCloudDD.TellUConsumer{
				event occurrence call_getItems1;
				then event occurrence return_getItems1;
				event occurrence call_getItems2;
				then event occurrence return_getItems2;
			}
			ref part apsp = AHFNorway_LocalCloudDD.APISProducer{
				event occurrence send_publish_returnallitems;
				then event occurrence receive_call_getItems1;
				then event occurrence send_returnallitems1;
				then event occurrence return_getItems_ack1;
				then event occurrence receive_call_getItems2;
				then event occurrence send_returnallitems2;
				then event occurrence return_getItems_ack2;
			}
			ref part mqtts = AHFNorway_LocalCloudDD.MQTTServer{
				event occurrence receive_publish_returnallitems;
				then event occurrence receive_subscribe_returnallitems;
				then event forw1:MQTTforwarding;
				then event forw2:MQTTforwarding;
			}
			ref part apsc = AHFNorway_LocalCloudDD.APISConsumer{
				event occurrence send_subscribe_returnallitems;
				then event forw1:MQTTforwarding;
				then event forw2:MQTTforwarding;
			}
			occurrence forw1:MQTTforwarding;	
			occurrence forw2:MQTTforwarding;	

			message publish_returnallitems of Publish
			from apsp.send_publish_returnallitems to mqtts.receive_publish_returnallitems;
			message subscribe_returnallitems of Subscribe
			from apsc.send_subscribe_returnallitems to mqtts.receive_subscribe_returnallitems;
			message call_getItems1 of CallGiveItems[1]
			from tlc.call_getItems1 to apsp.receive_call_getItems1;	
			bind apsp.send_returnallitems1 = forw1.mq; // binding the sending to the actual gate
			/* How to express that this event sends a Return_AllItems? */
			message returnack1 of ResultGiveItems
			from apsp.return_getItems_ack1 to tlc.return_getItems1;
			message call_getItems2 of CallGiveItems[1]
			from tlc.call_getItems2 to apsp.receive_call_getItems2;
			bind apsp.send_returnallitems2 = forw2.mq; // binding the sending to the actual gate
			message returnack2 of ResultGiveItems
			from apsp.return_getItems_ack2 to tlc.return_getItems2;
		}

		occurrence def MQTTforwarding {
			ref part mqttsf = AHFNorway_LocalCloudDD.MQTTServer{
				event occurrence receive_returnallitems;
				then event occurrence send_returnallitems;
			}

			ref part apscf :> AHFNorway_LocalCloudDD.APISConsumer {
				event occurrence receive_returnallitems;
			}

			in event occurrence mq; // parameter for gate

			message sendallitems1 of Return_AllItems
			from mq to mqttsf.receive_returnallitems;
			message sendallitems2 of Return_AllItems
			from mqttsf.send_returnallitems to apscf.receive_returnallitems;
		}

		
		interface APIS_transfer_interface : Interfaces::Interface connect (
			tlu ::> AHFNorway_LocalCloudDD.TellUConsumer.apisp.APIS_HTTP, // port reference
		    apsph ::> AHFNorway_LocalCloudDD.APISProducer.tellu.APIS_HTTP, 
			apspm ::> AHFNorway_LocalCloudDD.APISProducer.apisc.APIS_MQTT,
			apsc ::> AHFNorway_LocalCloudDD.APISConsumer.apisp.APIS_MQTT,
			mqget ::> AHFNorway_LocalCloudDD.MQTTServer.getTopic,
			mqgive ::> AHFNorway_LocalCloudDD.MQTTServer.giveTopic) {
			
			flow publish_returnallitems of Publish
			from apspm.pub to mqget.APIS_MQTT.pub;
			flow subscribe_returnallitems of Subscribe
			from apsc.subscr to mqgive.APIS_MQTT.subscr;
			flow call_getItems of CallGiveItems[1]
			from tlu.cll to apsph.cll;
			flow returnallitems of Return_AllItems
			from apspm.retall to mqget.APIS_MQTT.retall;
			flow sendallitems of Return_AllItems
			from mqgive.APIS_MQTT.retall to apsc.retall;
			flow returnack of ResultGiveItems
			from apsph.retrn to tlu.retrn;
			
			// Successions on each lifetime
			// tlu
			succession first call_getItems.start
			then returnack.done;	
			// apisp (taking both ports)
			succession first publish_returnallitems.start
			then call_getItems.done;
			succession first call_getItems.done
			then returnallitems.start;
			succession first returnallitems.start
			then returnack.start;
			// MQTTServer
			succession first publish_returnallitems.done
			then subscribe_returnallitems.done;
			succession first subscribe_returnallitems
			then returnallitems.done;
			succession first returnallitems.done
			then sendallitems.start;
			// apisc
			succession first subscribe_returnallitems.start
			then sendallitems.done;
		}
		
	}
}
~~~
# DIAGNOSTICS
~~~sexpr
(fixture-diagnostics
  (document "memory://snapshot/ahfsequences.md"
    (diagnostics
      (diagnostic
        (severity warning)
        (code "unresolved_import_target")
        (source "semantic")
        (range (start 3 16) (end 3 32))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_import_target")
        (source "semantic")
        (range (start 4 16) (end 4 29))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_import_target")
        (source "semantic")
        (range (start 5 16) (end 5 28))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_import_target")
        (source "semantic")
        (range (start 6 16) (end 6 31))
      )
      (diagnostic
        (severity warning)
        (code "unsupported_occurrence_definition_member")
        (source "semantic")
        (range (start 11 3) (end 16 4))
      )
      (diagnostic
        (severity warning)
        (code "unsupported_occurrence_definition_member")
        (source "semantic")
        (range (start 17 3) (end 25 4))
      )
      (diagnostic
        (severity warning)
        (code "unsupported_occurrence_definition_member")
        (source "semantic")
        (range (start 26 3) (end 31 4))
      )
      (diagnostic
        (severity warning)
        (code "unsupported_occurrence_definition_member")
        (source "semantic")
        (range (start 32 3) (end 36 4))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 40 37) (end 40 44))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 41 8) (end 41 40))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 41 44) (end 41 80))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 42 39) (end 42 48))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 43 8) (end 43 42))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 43 46) (end 43 84))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 44 29) (end 44 42))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 45 8) (end 45 26))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 45 30) (end 45 57))
      )
      (diagnostic
        (severity error)
        (code "unexpected_keyword_in_scope")
        (source "parser")
        (range (start 46 3) (end 48 3))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 48 25) (end 48 40))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 49 8) (end 49 33))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 49 37) (end 49 57))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 50 29) (end 50 42))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 51 8) (end 51 26))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 51 30) (end 51 57))
      )
      (diagnostic
        (severity error)
        (code "unexpected_keyword_in_scope")
        (source "parser")
        (range (start 52 3) (end 53 3))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 53 25) (end 53 40))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 54 8) (end 54 33))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 54 37) (end 54 57))
      )
      (diagnostic
        (severity warning)
        (code "unsupported_occurrence_definition_member")
        (source "semantic")
        (range (start 58 3) (end 61 4))
      )
      (diagnostic
        (severity warning)
        (code "unsupported_occurrence_definition_member")
        (source "semantic")
        (range (start 63 3) (end 65 4))
      )
      (diagnostic
        (severity warning)
        (code "unsupported_occurrence_definition_member")
        (source "semantic")
        (range (start 67 3) (end 67 26))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 69 28) (end 69 43))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 70 8) (end 70 10))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 70 14) (end 70 43))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 71 28) (end 71 43))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 72 8) (end 72 34))
      )
      (diagnostic
        (severity warning)
        (code "unresolved_reference")
        (source "semantic")
        (range (start 72 38) (end 72 66))
      )
      (diagnostic
        (severity error)
        (code "recovered_part_usage_body_element")
        (source "parser")
        (range (start 76 2) (end 120 1))
      )
    )
  )
)
~~~
# SMG
~~~sexpr
(semantic-model
  (publication (phase resolved) (completeness parse-recovery) (has-evaluation false) (source-digest "blake3:c523dfb40a40053478955a61e9a3269bf9070ed2e3c64c21d6ab58aa7c0b1ee6") (contract-version "parser-owned-resolution-v1"))
  (declarations
    (declaration (id (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences"))) (kind package) (membership (kind owning) (visibility default)))
    (declaration (id (node (document "memory://snapshot/ahfsequences.md") (anonymous (kind import) (ordinal 0))))) (kind import) (membership (kind import) (visibility private)) (authored (membership (kind import) (visibility private)) (relationships (namespaceImport (reference "AHFProfileLib") (import (shape namespace) (recursive false)))))
    (declaration (id (node (document "memory://snapshot/ahfsequences.md") (anonymous (kind import) (ordinal 1))))) (kind import) (membership (kind import) (visibility private)) (authored (membership (kind import) (visibility private)) (relationships (namespaceImport (reference "AHFCoreLib") (import (shape namespace) (recursive false)))))
    (declaration (id (node (document "memory://snapshot/ahfsequences.md") (anonymous (kind import) (ordinal 2))))) (kind import) (membership (kind import) (visibility private)) (authored (membership (kind import) (visibility private)) (relationships (namespaceImport (reference "AHFNorway") (import (shape namespace) (recursive false)))))
    (declaration (id (node (document "memory://snapshot/ahfsequences.md") (anonymous (kind import) (ordinal 3))))) (kind import) (membership (kind import) (visibility private)) (authored (membership (kind import) (visibility private)) (relationships (namespaceImport (reference "ScalarValues") (import (shape namespace) (recursive false)))))
    (declaration (id (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs"))) (kind part) (membership (kind feature) (visibility default)))
    (declaration (id (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime"))) (kind occurrence-def) (membership (kind owning) (visibility default)))
    (declaration (id (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::call_getItems1"))) (kind flow) (membership (kind feature) (visibility default)) (authored (membership (kind feature) (visibility default)) (relationships (memberAccessOperand (reference "tlc::call_getItems1")) (memberAccessOperand (reference "apsp::receive_call_getItems1")) (flowPayloadType (reference "CallGiveItems"))))
    (declaration (id (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::call_getItems2"))) (kind flow) (membership (kind feature) (visibility default)) (authored (membership (kind feature) (visibility default)) (relationships (memberAccessOperand (reference "tlc::call_getItems2")) (memberAccessOperand (reference "apsp::receive_call_getItems2")) (flowPayloadType (reference "CallGiveItems"))))
    (declaration (id (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::forw1"))) (kind occurrence) (membership (kind feature) (visibility default)) (authored (membership (kind feature) (visibility default)) (relationships (featureTyping (reference "MQTTforwarding"))))
    (declaration (id (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::forw2"))) (kind occurrence) (membership (kind feature) (visibility default)) (authored (membership (kind feature) (visibility default)) (relationships (featureTyping (reference "MQTTforwarding"))))
    (declaration (id (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::publish_returnallitems"))) (kind flow) (membership (kind feature) (visibility default)) (authored (membership (kind feature) (visibility default)) (relationships (memberAccessOperand (reference "apsp::send_publish_returnallitems")) (memberAccessOperand (reference "mqtts::receive_publish_returnallitems")) (flowPayloadType (reference "Publish"))))
    (declaration (id (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::returnack1"))) (kind flow) (membership (kind feature) (visibility default)) (authored (membership (kind feature) (visibility default)) (relationships (memberAccessOperand (reference "apsp::return_getItems_ack1")) (memberAccessOperand (reference "tlc::return_getItems1")) (flowPayloadType (reference "ResultGiveItems"))))
    (declaration (id (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::returnack2"))) (kind flow) (membership (kind feature) (visibility default)) (authored (membership (kind feature) (visibility default)) (relationships (memberAccessOperand (reference "apsp::return_getItems_ack2")) (memberAccessOperand (reference "tlc::return_getItems2")) (flowPayloadType (reference "ResultGiveItems"))))
    (declaration (id (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::subscribe_returnallitems"))) (kind flow) (membership (kind feature) (visibility default)) (authored (membership (kind feature) (visibility default)) (relationships (memberAccessOperand (reference "apsc::send_subscribe_returnallitems")) (memberAccessOperand (reference "mqtts::receive_subscribe_returnallitems")) (flowPayloadType (reference "Subscribe"))))
    (declaration (id (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::MQTTforwarding"))) (kind occurrence-def) (membership (kind owning) (visibility default)))
    (declaration (id (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::MQTTforwarding::sendallitems1"))) (kind flow) (membership (kind feature) (visibility default)) (authored (membership (kind feature) (visibility default)) (relationships (memberAccessOperand (reference "mqttsf::receive_returnallitems")) (flowSource (reference "mq")) (flowPayloadType (reference "Return_AllItems"))))
    (declaration (id (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::MQTTforwarding::sendallitems2"))) (kind flow) (membership (kind feature) (visibility default)) (authored (membership (kind feature) (visibility default)) (relationships (memberAccessOperand (reference "mqttsf::send_returnallitems")) (memberAccessOperand (reference "apscf::receive_returnallitems")) (flowPayloadType (reference "Return_AllItems"))))
  )
  (references
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (anonymous (kind import) (ordinal 0))))) (kind namespaceImport) (ordinal 0))
      (authored-target "AHFProfileLib")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (anonymous (kind import) (ordinal 1))))) (kind namespaceImport) (ordinal 0))
      (authored-target "AHFCoreLib")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (anonymous (kind import) (ordinal 2))))) (kind namespaceImport) (ordinal 0))
      (authored-target "AHFNorway")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (anonymous (kind import) (ordinal 3))))) (kind namespaceImport) (ordinal 0))
      (authored-target "ScalarValues")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::call_getItems1"))) (kind memberAccessOperand) (ordinal 0))
      (authored-target "tlc::call_getItems1")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::call_getItems1"))) (kind memberAccessOperand) (ordinal 1))
      (authored-target "apsp::receive_call_getItems1")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::call_getItems1"))) (kind flowPayloadType) (ordinal 0))
      (authored-target "CallGiveItems")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::call_getItems2"))) (kind memberAccessOperand) (ordinal 0))
      (authored-target "tlc::call_getItems2")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::call_getItems2"))) (kind memberAccessOperand) (ordinal 1))
      (authored-target "apsp::receive_call_getItems2")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::call_getItems2"))) (kind flowPayloadType) (ordinal 0))
      (authored-target "CallGiveItems")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::forw1"))) (kind featureTyping) (ordinal 0))
      (authored-target "MQTTforwarding")
      (outcome (status resolved) (target (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::MQTTforwarding")))))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::forw2"))) (kind featureTyping) (ordinal 0))
      (authored-target "MQTTforwarding")
      (outcome (status resolved) (target (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::MQTTforwarding")))))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::publish_returnallitems"))) (kind memberAccessOperand) (ordinal 0))
      (authored-target "apsp::send_publish_returnallitems")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::publish_returnallitems"))) (kind memberAccessOperand) (ordinal 1))
      (authored-target "mqtts::receive_publish_returnallitems")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::publish_returnallitems"))) (kind flowPayloadType) (ordinal 0))
      (authored-target "Publish")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::returnack1"))) (kind memberAccessOperand) (ordinal 0))
      (authored-target "apsp::return_getItems_ack1")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::returnack1"))) (kind memberAccessOperand) (ordinal 1))
      (authored-target "tlc::return_getItems1")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::returnack1"))) (kind flowPayloadType) (ordinal 0))
      (authored-target "ResultGiveItems")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::returnack2"))) (kind memberAccessOperand) (ordinal 0))
      (authored-target "apsp::return_getItems_ack2")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::returnack2"))) (kind memberAccessOperand) (ordinal 1))
      (authored-target "tlc::return_getItems2")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::returnack2"))) (kind flowPayloadType) (ordinal 0))
      (authored-target "ResultGiveItems")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::subscribe_returnallitems"))) (kind memberAccessOperand) (ordinal 0))
      (authored-target "apsc::send_subscribe_returnallitems")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::subscribe_returnallitems"))) (kind memberAccessOperand) (ordinal 1))
      (authored-target "mqtts::receive_subscribe_returnallitems")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::subscribe_returnallitems"))) (kind flowPayloadType) (ordinal 0))
      (authored-target "Subscribe")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::MQTTforwarding::sendallitems1"))) (kind memberAccessOperand) (ordinal 0))
      (authored-target "mqttsf::receive_returnallitems")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::MQTTforwarding::sendallitems1"))) (kind flowSource) (ordinal 0))
      (authored-target "mq")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::MQTTforwarding::sendallitems1"))) (kind flowPayloadType) (ordinal 0))
      (authored-target "Return_AllItems")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::MQTTforwarding::sendallitems2"))) (kind memberAccessOperand) (ordinal 0))
      (authored-target "mqttsf::send_returnallitems")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::MQTTforwarding::sendallitems2"))) (kind memberAccessOperand) (ordinal 1))
      (authored-target "apscf::receive_returnallitems")
      (outcome (status unresolved)))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::MQTTforwarding::sendallitems2"))) (kind flowPayloadType) (ordinal 0))
      (authored-target "Return_AllItems")
      (outcome (status unresolved)))
  )
  (relationships
    (relationship (kind typing) (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::forw1"))) (target (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::MQTTforwarding"))) (provenance authored) (authored-reference (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::forw1"))) (kind featureTyping) (ordinal 0)))
    (relationship (kind typing) (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::forw2"))) (target (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::MQTTforwarding"))) (provenance authored) (authored-reference (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::forw2"))) (kind featureTyping) (ordinal 0)))
  )
  (evaluation
  )
)
~~~
# NAVIGATION
~~~sexpr
(navigation
  (query (document "memory://snapshot/ahfsequences.md") (range (start 3 16) (end 3 32)) (probe (position 3 16))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (anonymous (kind import) (ordinal 0))))) (kind namespaceImport) (ordinal 0) (authored-target "AHFProfileLib")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 4 16) (end 4 29)) (probe (position 4 16))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (anonymous (kind import) (ordinal 1))))) (kind namespaceImport) (ordinal 0) (authored-target "AHFCoreLib")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 5 16) (end 5 28)) (probe (position 5 16))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (anonymous (kind import) (ordinal 2))))) (kind namespaceImport) (ordinal 0) (authored-target "AHFNorway")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 6 16) (end 6 31)) (probe (position 6 16))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (anonymous (kind import) (ordinal 3))))) (kind namespaceImport) (ordinal 0) (authored-target "ScalarValues")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 45 8) (end 45 26)) (probe (position 45 8))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::call_getItems1"))) (kind memberAccessOperand) (ordinal 0) (authored-target "tlc::call_getItems1")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 45 30) (end 45 57)) (probe (position 45 30))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::call_getItems1"))) (kind memberAccessOperand) (ordinal 1) (authored-target "apsp::receive_call_getItems1")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 44 29) (end 44 42)) (probe (position 44 29))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::call_getItems1"))) (kind flowPayloadType) (ordinal 0) (authored-target "CallGiveItems")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 51 8) (end 51 26)) (probe (position 51 8))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::call_getItems2"))) (kind memberAccessOperand) (ordinal 0) (authored-target "tlc::call_getItems2")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 51 30) (end 51 57)) (probe (position 51 30))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::call_getItems2"))) (kind memberAccessOperand) (ordinal 1) (authored-target "apsp::receive_call_getItems2")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 50 29) (end 50 42)) (probe (position 50 29))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::call_getItems2"))) (kind flowPayloadType) (ordinal 0) (authored-target "CallGiveItems")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 37 20) (end 37 34)) (probe (position 37 20))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::forw1"))) (kind featureTyping) (ordinal 0) (authored-target "MQTTforwarding")
      (outcome (status resolved) (target (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::MQTTforwarding")))))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 38 20) (end 38 34)) (probe (position 38 20))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::forw2"))) (kind featureTyping) (ordinal 0) (authored-target "MQTTforwarding")
      (outcome (status resolved) (target (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::MQTTforwarding")))))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 41 8) (end 41 40)) (probe (position 41 8))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::publish_returnallitems"))) (kind memberAccessOperand) (ordinal 0) (authored-target "apsp::send_publish_returnallitems")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 41 44) (end 41 80)) (probe (position 41 44))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::publish_returnallitems"))) (kind memberAccessOperand) (ordinal 1) (authored-target "mqtts::receive_publish_returnallitems")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 40 37) (end 40 44)) (probe (position 40 37))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::publish_returnallitems"))) (kind flowPayloadType) (ordinal 0) (authored-target "Publish")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 49 8) (end 49 33)) (probe (position 49 8))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::returnack1"))) (kind memberAccessOperand) (ordinal 0) (authored-target "apsp::return_getItems_ack1")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 49 37) (end 49 57)) (probe (position 49 37))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::returnack1"))) (kind memberAccessOperand) (ordinal 1) (authored-target "tlc::return_getItems1")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 48 25) (end 48 40)) (probe (position 48 25))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::returnack1"))) (kind flowPayloadType) (ordinal 0) (authored-target "ResultGiveItems")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 54 8) (end 54 33)) (probe (position 54 8))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::returnack2"))) (kind memberAccessOperand) (ordinal 0) (authored-target "apsp::return_getItems_ack2")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 54 37) (end 54 57)) (probe (position 54 37))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::returnack2"))) (kind memberAccessOperand) (ordinal 1) (authored-target "tlc::return_getItems2")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 53 25) (end 53 40)) (probe (position 53 25))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::returnack2"))) (kind flowPayloadType) (ordinal 0) (authored-target "ResultGiveItems")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 43 8) (end 43 42)) (probe (position 43 8))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::subscribe_returnallitems"))) (kind memberAccessOperand) (ordinal 0) (authored-target "apsc::send_subscribe_returnallitems")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 43 46) (end 43 84)) (probe (position 43 46))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::subscribe_returnallitems"))) (kind memberAccessOperand) (ordinal 1) (authored-target "mqtts::receive_subscribe_returnallitems")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 42 39) (end 42 48)) (probe (position 42 39))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::APIS_transfer_lifetime::subscribe_returnallitems"))) (kind flowPayloadType) (ordinal 0) (authored-target "Subscribe")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 70 14) (end 70 43)) (probe (position 70 14))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::MQTTforwarding::sendallitems1"))) (kind memberAccessOperand) (ordinal 0) (authored-target "mqttsf::receive_returnallitems")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 70 8) (end 70 10)) (probe (position 70 8))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::MQTTforwarding::sendallitems1"))) (kind flowSource) (ordinal 0) (authored-target "mq")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 69 28) (end 69 43)) (probe (position 69 28))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::MQTTforwarding::sendallitems1"))) (kind flowPayloadType) (ordinal 0) (authored-target "Return_AllItems")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 72 8) (end 72 34)) (probe (position 72 8))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::MQTTforwarding::sendallitems2"))) (kind memberAccessOperand) (ordinal 0) (authored-target "mqttsf::send_returnallitems")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 72 38) (end 72 66)) (probe (position 72 38))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::MQTTforwarding::sendallitems2"))) (kind memberAccessOperand) (ordinal 1) (authored-target "apscf::receive_returnallitems")
      (outcome (status unresolved)))
  )
  (query (document "memory://snapshot/ahfsequences.md") (range (start 71 28) (end 71 43)) (probe (position 71 28))
    (reference (id (source (node (document "memory://snapshot/ahfsequences.md") (qualified-name "AHFNorwaySequences::AHFN_LocalCloudDD_Seqs::MQTTforwarding::sendallitems2"))) (kind flowPayloadType) (ordinal 0) (authored-target "Return_AllItems")
      (outcome (status unresolved)))
  )
)
~~~
