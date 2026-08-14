{
  "Comment": "An example of using Bedrock to chain prompts and their responses together.",
  "StartAt": "Invoke model with firts prompt",
  "QueryLanguage": "JSONata",
  "States": {
    "Invoke model with firts prompt": {
      "Type": "Task",
      "Resource": "arn:aws:states:::bedrock:invokeModel",
      "Arguments": {
        "ModelId": "arn:aws:bedrock:us-east-2::foundation-model/amazon.titan-embed-text-v2:0",
        "Body": {
          "inputText": "estou programando um jantar romantico, nesse jantar vou pedir um ossobuco, me de uma lista de 3 items que combinem em uma experiência gastronomica",
          "textGenerationConfig": {
            "temperature": 0,
            "topP": 1,
            "maxTokenCount": 512
          }
        },
        "ContentType": "application/json",
        "Accept": "application/json"
      },
      "Next": "add first result to convesation history",
      "Output": {
        "result_one.$": "$.Body.inputText[0]"
      }
    },
    "add first result to convesation history": {
      "Type": "Pass",
      "Next": "invoke model with second promt",
      "Output": {
        "convo_one.$": "States.Format('{}\n{}',$.prompt_one,$.result_one)"
      },
      "Assign": {
        "counter": "{% $count($states.input.prompts) %}",
        "conversation_history": [
          ""
        ],
        "input_prompts": "{% $reverse($states.input.prompts) %}"
      }
    },
    "invoke model with second promt": {
      "Type": "Task",
      "Resource": "arn:aws:states:::bedrock:invokeModel",
      "Arguments": {
        "ModelId": "arn:aws:bedrock:us-east-2::foundation-model/amazon.titan-embed-text-v2:0",
        "Body": {
          "inputText": "liste duas bebidas que acompanhem um jantar romantico",
          "textGenerationConfig": {
            "temperature": 0,
            "topP": 1,
            "maxTokenCount": 512
          }
        },
        "ContentType": "application/json",
        "Accept": "application/json"
      },
      "Assign": {
        "conversation_history": "{% $append($conversation_history, $states.result.Body.output.message.content[0].text) %}",
        "counter": "{% $counter - 1 %}"
      },
      "Next": "add second result to convesation history",
      "Output": {
        "result_two.$": "$.Body.inputText[0]"
      }
    },
    "add second result to convesation history": {
      "Type": "Pass",
      "Next": "add third result to convesation history",
      "Output": {
        "convo_two.$": "States.Format('{}\n{}', $.convo_one, $.prompt_two, $.result_two)"
      },
      "Assign": {
        "counter": "{% $count($states.input.prompts) %}",
        "conversation_history": [
          ""
        ],
        "input_prompts": "{% $reverse($states.input.prompts) %}"
      }
    },
    "add third result to convesation history": {
      "Type": "Task",
      "Resource": "arn:aws:states:::bedrock:invokeModel",
      "Arguments": {
        "ModelId": "arn:aws:bedrock:us-east-2::foundation-model/amazon.titan-embed-text-v2:0",
        "Body": {
          "inputText": "informe um lugar perfeito para jantar romantico em paris",
          "textGenerationConfig": {
            "temperature": 0,
            "topP": 1,
            "maxTokenCount": 512
          }
        },
        "ContentType": "application/json",
        "Accept": "application/json"
      },
      "Assign": {
        "conversation_history": "{% $append($conversation_history, $states.result.Body.output.message.content[0].text) %}",
        "counter": "{% $counter - 1 %}"
      },
      "End": true,
      "Output": {
        "result_three.$": "$.Body.inputText[0]"
      }
    }
  }
}
