<!DOCTYPE html>
.journal-container {
  max-width: 1000px;
  margin: 30px auto;
  font-family: Arial, sans-serif;
}
.journal-header {
  font-size: 22px;
  margin-bottom: 10px;
}
.journal-toolbar {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 10px;
}
.journal-toolbar-btn {
  background: #1976d2;
  color: #fff;
  border: none;
  border-radius: 5px;
  padding: 8px 14px;
  font-size: 15px;
  cursor: pointer;
}
.journal-error {
  background: #ff5b5b;
  color: white;
  padding: 10px 16px;
  border-radius: 8px;
  margin-bottom: 10px;
  font-size: 18px;
  display: flex;
  align-items: center;
  gap: 9px;
}
.journal-error-icon {
  font-size: 22px;
  margin-right: 6px;
  font-weight: bold;
}
.journal-table {
  width: 100%;
  border-collapse: collapse;
  background: #fff;
  box-shadow: 0 2px 8px rgba(0,0,0,0.05);
}
.journal-table th, .journal-table td {
  border: 1px solid #e2e2e2;
  padding: 6px 7px;
  text-align: center;
}
.journal-input {
  width: 36px;
  height: 32px;
  font-size: 18px;
  text-align: center;
  border: 1px solid #ccc;
  border-radius: 5px;
  transition: border 0.2s;
}
.journal-input:focus {
  border: 1.5px solid #1976d2;
  outline: none;
}
.journal-absence {
  font-size: 18px;
  background: #f7f9fa;
}
.journal-tooltip {
  position: fixed;
  background: #082c4c;
  color: #fff;
  padding: 6px 15px;
  border-radius: 8px;
  font-size: 16px;
  z-index: 9999;
  white-space: nowrap;
  box-shadow: 0 2px 8px rgba(0,0,0,.09);
  pointer-events: none;
}
import React, { useState } from "react";
import "./ElectronicJournal.css";

const students = [
  { id: 1, name: "Иванов Иван" },
  { id: 2, name: "Петров Петр" },
  { id: 3, name: "Сидоров Сидор" },
];

const lessons = [
  "Инф. безопасность",
  "Математические основы информатики",
  "Алгоритмы",
  "Дискретная математика",
];

const absenceCodes = {
  "Б": { color: "#2ecc40", tooltip: "Болеет" },
  "Н": { color: "#ff4136", tooltip: "Неуваж. пропуск" },
  "П": { color: "#0074d9", tooltip: "Уваж. пропуск" },
};

function isAbsence(val) {
  return ["Б", "Н", "П"].includes(val);
}

export default function ElectronicJournal() {
  const [grades, setGrades] = useState(() =>
    students.map(() =>
      lessons.map(() => "")
    )
  );
  const [error, setError] = useState("");

  const handleCellChange = (studentIdx, lessonIdx, value) => {
    let newGrades = grades.map(arr => arr.slice());
    // 1-5 or letter
    if (value.match(/^[1-5]$/)) {
      newGrades[studentIdx][lessonIdx] = value;
      setGrades(newGrades);
      setError(""); // reset error if correct
    } else if (isAbsence(value)) {
      newGrades[studentIdx][lessonIdx] = value;
      setGrades(newGrades);
      setError("");
    } else if (value) {
      newGrades[studentIdx][lessonIdx] = value;
      setGrades(newGrades);
      setError("Неправильная заполняемость журнала");
    } else {
      newGrades[studentIdx][lessonIdx] = "";
      setGrades(newGrades);
      setError("");
    }
  };

  // Check for any value above 5
  React.useEffect(() => {
    for (let row of grades)
      for (let val of row)
        if (val.match(/^[6-9]$|^[1-9][0-9]+$/)) {
          setError("Неправильная заполняемость журнала");
          return;
        }
  }, [grades]);

  return (
    <div className="journal-container">
      <div className="journal-header">
        Журнал Информатики 10А
      </div>
      <div className="journal-toolbar">
        <button className="journal-toolbar-btn">Отметки/отсутствия</button>
      </div>
      {error && (
        <div className="journal-error">
          <span className="journal-error-icon">!</span>
          {error}
        </div>
      )}
      <table className="journal-table">
        <thead>
          <tr>
            <th>№</th>
            <th>ФИО</th>
            {lessons.map(lesson => (
              <th key={lesson}>{lesson}</th>
            ))}
          </tr>
        </thead>
        <tbody>
          {students.map((student, studentIdx) => (
            <tr key={student.id}>
              <td>{studentIdx + 1}</td>
              <td>{student.name}</td>
              {lessons.map((lesson, lessonIdx) => {
                const val = grades[studentIdx][lessonIdx];
                const abs = isAbsence(val) ? absenceCodes[val] : null;
                return (
                  <td key={lessonIdx} className="journal-cell">
                    <input
                      className={`journal-input${abs ? " journal-absence" : ""}`}
                      style={abs ? { color: abs.color, fontWeight: "bold" } : {}}
                      type="text"
                      maxLength={1}
                      value={val}
                      onChange={e => handleCellChange(studentIdx, lessonIdx, e.target.value.toUpperCase())}
                      onMouseOver={e => {
                        if (abs) {
                          const tooltip = document.createElement("div");
                          tooltip.className = "journal-tooltip";
                          tooltip.innerText = abs.tooltip;
                          document.body.appendChild(tooltip);
                          const rect = e.target.getBoundingClientRect();
                          tooltip.style.left = rect.left + "px";
                          tooltip.style.top = rect.bottom + 5 + "px";
                          e.target._tooltip = tooltip;
                        }
                      }}
                      onMouseOut={e => {
                        if (e.target._tooltip) {
                          document.body.removeChild(e.target._tooltip);
                          e.target._tooltip = null;
                        }
                      }}
                    />
                  </td>
                );
              })}
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
