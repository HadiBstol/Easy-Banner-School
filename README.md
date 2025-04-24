import React, { useState, useRef } from "react";
import { Button } from "@/components/ui/button";
import jsPDF from "jspdf";

export default function RasterbatorDemo() {
  const [image, setImage] = useState(null);
  const [columns, setColumns] = useState(3);
  const [rows, setRows] = useState(2);
  const [paperSize, setPaperSize] = useState("A4");
  const canvasRef = useRef();

  const handleImageUpload = (e) => {
    const file = e.target.files[0];
    if (file) {
      const reader = new FileReader();
      reader.onloadend = () => {
        setImage(reader.result);
      };
      reader.readAsDataURL(file);
    }
  };

  const getPageDimensions = () => {
    if (paperSize === "A4") return [210, 297];
    if (paperSize === "A3") return [297, 420];
    return [210, 297];
  };

  const generatePDF = async () => {
    const img = new Image();
    img.src = image;
    await img.decode();

    const [pageWidth, pageHeight] = getPageDimensions();
    const pdf = new jsPDF({ orientation: "portrait", unit: "mm", format: paperSize });

    const sliceWidth = img.width / columns;
    const sliceHeight = img.height / rows;

    const canvas = document.createElement("canvas");
    const ctx = canvas.getContext("2d");
    canvas.width = sliceWidth;
    canvas.height = sliceHeight;

    for (let y = 0; y < rows; y++) {
      for (let x = 0; x < columns; x++) {
        ctx.clearRect(0, 0, sliceWidth, sliceHeight);
        ctx.drawImage(
          img,
          x * sliceWidth,
          y * sliceHeight,
          sliceWidth,
          sliceHeight,
          0,
          0,
          sliceWidth,
          sliceHeight
        );

        const imgData = canvas.toDataURL("image/jpeg", 1.0);
        if (x !== 0 || y !== 0) pdf.addPage();
        pdf.addImage(imgData, "JPEG", 0, 0, pageWidth, pageHeight);
      }
    }

    pdf.save("rasterbated-poster.pdf");
  };

  return (
    <div className="p-6 max-w-3xl mx-auto">
      <h1 className="text-2xl font-bold mb-4">Mini Rasterbator Demo</h1>

      <input type="file" accept="image/*" onChange={handleImageUpload} className="mb-4" />

      {image && (
        <div className="border p-2 mb-4">
          <img src={image} alt="Preview" className="max-w-full h-auto" />
        </div>
      )}

      <div className="mb-4">
        <label className="mr-2">Columns:</label>
        <input
          type="number"
          value={columns}
          onChange={(e) => setColumns(Number(e.target.value))}
          className="border p-1 w-16 mr-4"
        />

        <label className="mr-2">Rows:</label>
        <input
          type="number"
          value={rows}
          onChange={(e) => setRows(Number(e.target.value))}
          className="border p-1 w-16 mr-4"
        />

        <label className="mr-2">Paper Size:</label>
        <select
          value={paperSize}
          onChange={(e) => setPaperSize(e.target.value)}
          className="border p-1"
        >
          <option value="A4">A4</option>
          <option value="A3">A3</option>
        </select>
      </div>

      <Button onClick={generatePDF}>Generate PDF</Button>
    </div>
  );
}
