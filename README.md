import React, { useState } from 'react'; import { Button } from "@/components/ui/button"; import { Card, CardContent } from "@/components/ui/card"; import { Heatmap } from "lucide-react";

const generateHeatmap = () => { const grid = Array(16).fill(0).map(() => Array(16).fill(0)); const topBits = [34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 46, 47, 48, 49, 62]; for (const i of topBits) { const row = Math.floor(i / 16); const col = i % 16; grid[row][col] = 1; } return grid; };

export default function NonceChessGame() { const [heatmap, setHeatmap] = useState(generateHeatmap()); const [result, setResult] = useState(null);

const simulateNonce = () => { const bits = Array(256).fill(0); const topBits = [34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 46, 47, 48, 49, 62]; for (const i of topBits) bits[i] = Math.random() < 0.5 ? 1 : 0; const hex = parseInt(bits.join(""), 2).toString(16).padStart(64, '0'); setResult(hex.slice(0, 8)); };

return ( <div className="p-4 space-y-6"> <Card> <CardContent className="grid grid-cols-16 gap-1 p-4"> {heatmap.flat().map((val, i) => ( <div key={i} className={w-4 h-4 rounded ${val ? 'bg-blue-500' : 'bg-gray-300'}} /> ))} </CardContent> </Card>

<div className="flex gap-4">
    <Button onClick={simulateNonce}>
      🔁 جرّب نونس ذكي
    </Button>
    {result && (
      <div className="text-lg font-mono">🧬 نونس: <span className="text-green-600">{result}</span></div>
    )}
  </div>
</div>

); }

