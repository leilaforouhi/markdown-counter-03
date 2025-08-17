#!/usr/bin/env python3
"""
Text & Code Stats Reporter
- Scans the current repository (recursively) for text-like files
- Computes lines/words/chars per file.
- Saves a machine-readable summary to report.json and prints a short summary.
"""

from __future__ import annotations
import json, os, sys, datetime

TEXT_EXTS = {
    ".txt", ".md", ".rst", ".py", ".js", ".ts", ".tsx", ".jsx",
    ".json", ".yml", ".yaml", ".html", ".css", ".scss", ".toml",
    ".ini", ".cfg", ".sh", ".bat", ".ps1", ".java", ".kt", ".go",
    ".rb", ".php", ".pl", ".c", ".h", ".cpp", ".hpp", ".cs", ".swift"
}

def is_texty(path: str) -> bool:
    _, ext = os.path.splitext(path.lower())
    return ext in TEXT_EXTS

def file_stats(path: str) -> dict:
    try:
        with open(path, "r", encoding="utf-8", errors="ignore") as f:
            content = f.read()
    except Exception as e:
        return {"path": path, "error": str(e)}

    lines = content.splitlines()
    words = content.split()
    return {
        "path": path,
        "lines": len(lines),
        "words": len(words),
        "chars": len(content),
    }

def main(root: str = ".") -> int:
    files = []
    for base, _, names in os.walk(root):
        # skip hidden folders like .git
        parts = os.path.normpath(base).split(os.sep)
        if any(p.startswith(".") for p in parts):
            continue
        for n in names:
            p = os.path.join(base, n)
            if is_texty(p):
                files.append(p)

    stats = [file_stats(p) for p in sorted(files)]
    totals = {
        "files": sum(1 for s in stats if "error" not in s),
        "lines": sum(s.get("lines", 0) for s in stats),
        "words": sum(s.get("words", 0) for s in stats),
        "chars": sum(s.get("chars", 0) for s in stats),
    }

    report = {
        "generated_at": datetime.datetime.now().isoformat(timespec="seconds"),
        "root": os.path.abspath(root),
        "totals": totals,
        "files": stats,
    }

    with open("report.json", "w", encoding="utf-8") as f:
        json.dump(report, f, indent=2, ensure_ascii=False)

    print("Report saved to report.json")
    print(f"Files: {totals['files']} | Lines: {totals['lines']} | Words: {totals['words']} | Chars: {totals['chars']}")
    return 0

if __name__ == "__main__":
    sys.exit(main("."))
