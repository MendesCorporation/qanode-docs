# Provider Examples

Examples of custom node providers in various programming languages.

---

## Node.js (Express)

```javascript
import express from 'express';

const app = express();
app.use(express.json());

app.get('/manifest', (req, res) => {
  res.json({
    nodes: [{
      type: 'custom-echo',
      name: 'Echo',
      category: 'Custom Nodes',
      inputSchema: {
        message: { type: 'string', required: true }
      },
      outputSchema: {
        echo: { type: 'string' },
        timestamp: { type: 'string' }
      }
    }]
  });
});

app.post('/execute', (req, res) => {
  const { nodeType, inputs } = req.body;

  if (nodeType === 'custom-echo') {
    res.json({
      status: 'success',
      logs: [`Echoing: "${inputs.message}"`],
      outputs: {
        echo: inputs.message,
        timestamp: new Date().toISOString()
      },
      artifacts: []
    });
  }
});

app.listen(4010);
```

---

## Python (Flask)

```python
from flask import Flask, request, jsonify
from datetime import datetime

app = Flask(__name__)

@app.route("/manifest", methods=["GET"])
def manifest():
    return jsonify({
        "nodes": [{
            "type": "python-echo",
            "name": "Python Echo",
            "category": "Custom (Python)",
            "inputSchema": {
                "message": {"type": "string", "required": True}
            },
            "outputSchema": {
                "echo": {"type": "string"},
                "timestamp": {"type": "string"}
            }
        }]
    })

@app.route("/execute", methods=["POST"])
def execute():
    data = request.json
    node_type = data["nodeType"]
    inputs = data["inputs"]

    if node_type == "python-echo":
        return jsonify({
            "status": "success",
            "logs": [f'Echoing: "{inputs["message"]}"'],
            "outputs": {
                "echo": inputs["message"],
                "timestamp": datetime.now().isoformat()
            },
            "artifacts": []
        })

    return jsonify({
        "status": "failed",
        "error": {"message": f"Unknown node: {node_type}"}
    })

if __name__ == "__main__":
    app.run(port=4010)
```

---

## Python (FastAPI)

```python
from fastapi import FastAPI
from pydantic import BaseModel
from datetime import datetime
from typing import Any

app = FastAPI()

class ExecuteRequest(BaseModel):
    nodeType: str
    inputs: dict[str, Any]
    runId: str = ""
    nodeId: str = ""

@app.get("/manifest")
def manifest():
    return {
        "nodes": [{
            "type": "fast-echo",
            "name": "FastAPI Echo",
            "category": "Custom (Python)",
            "inputSchema": {
                "message": {"type": "string", "required": True}
            },
            "outputSchema": {
                "echo": {"type": "string"}
            }
        }]
    }

@app.post("/execute")
def execute(req: ExecuteRequest):
    if req.nodeType == "fast-echo":
        return {
            "status": "success",
            "logs": [f"Echo: {req.inputs['message']}"],
            "outputs": {"echo": req.inputs["message"]},
            "artifacts": []
        }
    return {"status": "failed", "error": {"message": "Unknown node"}}
```

---

## Java (Spring Boot)

```java
@RestController
public class ProviderController {

    @GetMapping("/manifest")
    public Map<String, Object> manifest() {
        Map<String, Object> node = new HashMap<>();
        node.put("type", "java-echo");
        node.put("name", "Java Echo");
        node.put("category", "Custom (Java)");
        node.put("inputSchema", Map.of(
            "message", Map.of("type", "string", "required", true)
        ));
        node.put("outputSchema", Map.of(
            "echo", Map.of("type", "string")
        ));

        return Map.of("nodes", List.of(node));
    }

    @PostMapping("/execute")
    public Map<String, Object> execute(@RequestBody Map<String, Object> body) {
        String nodeType = (String) body.get("nodeType");
        Map<String, Object> inputs = (Map<String, Object>) body.get("inputs");

        if ("java-echo".equals(nodeType)) {
            return Map.of(
                "status", "success",
                "logs", List.of("Echo: " + inputs.get("message")),
                "outputs", Map.of("echo", inputs.get("message")),
                "artifacts", List.of()
            );
        }

        return Map.of(
            "status", "failed",
            "error", Map.of("message", "Unknown node: " + nodeType)
        );
    }
}
```

---

## C# (ASP.NET Core)

```csharp
app.MapGet("/manifest", () => new {
    nodes = new[] {
        new {
            type = "csharp-echo",
            name = "C# Echo",
            category = "Custom (C#)",
            inputSchema = new {
                message = new { type = "string", required = true }
            },
            outputSchema = new {
                echo = new { type = "string" }
            }
        }
    }
});

app.MapPost("/execute", (ExecuteRequest req) => {
    if (req.NodeType == "csharp-echo") {
        return Results.Json(new {
            status = "success",
            logs = new[] { $"Echo: {req.Inputs["message"]}" },
            outputs = new { echo = req.Inputs["message"] },
            artifacts = Array.Empty<object>()
        });
    }
    return Results.Json(new {
        status = "failed",
        error = new { message = $"Unknown node: {req.NodeType}" }
    });
});

record ExecuteRequest(string NodeType, Dictionary<string, object> Inputs, string RunId, string NodeId);
```

---

## Go

```go
package main

import (
    "encoding/json"
    "net/http"
)

func main() {
    http.HandleFunc("/manifest", func(w http.ResponseWriter, r *http.Request) {
        json.NewEncoder(w).Encode(map[string]interface{}{
            "nodes": []map[string]interface{}{
                {
                    "type":     "go-echo",
                    "name":     "Go Echo",
                    "category": "Custom (Go)",
                    "inputSchema": map[string]interface{}{
                        "message": map[string]interface{}{"type": "string", "required": true},
                    },
                    "outputSchema": map[string]interface{}{
                        "echo": map[string]interface{}{"type": "string"},
                    },
                },
            },
        })
    })

    http.HandleFunc("/execute", func(w http.ResponseWriter, r *http.Request) {
        var req struct {
            NodeType string                 `json:"nodeType"`
            Inputs   map[string]interface{} `json:"inputs"`
        }
        json.NewDecoder(r.Body).Decode(&req)

        if req.NodeType == "go-echo" {
            json.NewEncoder(w).Encode(map[string]interface{}{
                "status":    "success",
                "logs":      []string{"Echo: " + req.Inputs["message"].(string)},
                "outputs":   map[string]interface{}{"echo": req.Inputs["message"]},
                "artifacts": []interface{}{},
            })
            return
        }
        json.NewEncoder(w).Encode(map[string]interface{}{
            "status": "failed",
            "error":  map[string]string{"message": "Unknown node"},
        })
    })

    http.ListenAndServe(":4010", nil)
}
```

---

## Advanced Example: Node with Artifact (Screenshot)

```javascript
// gerar-badge/gerar-badge.node.js
import { createCanvas } from 'canvas';

export const manifest = {
  type: 'gerar-badge',
  name: 'Gerar Badge',
  category: 'Gerador',
  inputSchema: {
    texto: { type: 'string', required: true },
    cor: { type: 'string', enum: ['green', 'red', 'blue', 'yellow'], default: 'green' }
  },
  outputSchema: {
    imageName: { type: 'string' }
  }
};

export async function execute({ inputs }) {
  const canvas = createCanvas(200, 40);
  const ctx = canvas.getContext('2d');

  // Draw the badge
  ctx.fillStyle = inputs.cor || 'green';
  ctx.fillRect(0, 0, 200, 40);
  ctx.fillStyle = 'white';
  ctx.font = 'bold 16px sans-serif';
  ctx.textAlign = 'center';
  ctx.fillText(inputs.texto, 100, 26);

  const base64 = canvas.toBuffer('image/png').toString('base64');

  return {
    status: 'success',
    logs: [`Badge gerado: "${inputs.texto}" (${inputs.cor})`],
    outputs: { imageName: `badge-${Date.now()}.png` },
    artifacts: [{
      type: 'screenshot',
      name: `badge-${Date.now()}.png`,
      base64
    }]
  };
}
```

---

## Tips

- Start with the simplest example (echo) and expand gradually
- Use the `*.node.js` convention for node files and **never** use `.node.` in auxiliary files
- Organize into **one folder per node** with its own `package.json` (see [Creating a Provider](creating-enterprise-provider.md))
- Use the **health check** to monitor whether the provider is active
- Implement **detailed logs** — they appear in the execution results
- For production providers, add **authentication** via Bearer token
- Consider using **Docker** to simplify provider deployment
