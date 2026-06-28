v1/v2 https://claude.ai/share/42a4b7bc-33a3-4064-a52b-54531259089a



Docker, Traffic: 
https://claude.ai/share/c438c572-f4d6-4953-976e-b655d23cc145


```java
@echo off
REM ============================================================
REM  check-release.bat
REM  Prueft, ob der HOECHSTE Release-Branch (z.B. release/1.60)
REM  inhaltlich vollstaendig im Ziel-Branch (master) enthalten ist.
REM  Vergleich ueber 'git cherry' (patch-id) - nicht ueber SHAs,
REM  d.h. cherry-gepickte Fixes werden korrekt als vorhanden erkannt.
REM
REM  Ergebnis / Exit-Code:
REM    0 = OK    (nichts fehlt)
REM    1 = FEHLT (mind. ein Release-Fix ist nicht in master)
REM    2 = FEHLER (kein Release-Branch / kein Git-Repo)
REM ============================================================
setlocal enabledelayedexpansion

REM ins Verzeichnis dieser .bat wechseln (= das Git-Repo)
cd /d "%~dp0"

REM ----------- Konfiguration: an eure Benennung anpassen -----------
set "TARGET=master"
set "RELEASE_GLOB=refs/remotes/origin/release/*"
REM   Beispiel, falls eure Branches 'Release-Branch-1.59' heissen:
REM   set "RELEASE_GLOB=refs/remotes/origin/Release-Branch-*"
REM ----------------------------------------------------------------

echo ============================================================
echo  Release-in-Master Check
echo ============================================================

REM ---- aktuellen Stand holen (Fehler nicht fatal) ----
git fetch --quiet --prune origin
if errorlevel 1 echo WARNUNG: 'git fetch' fehlgeschlagen - nutze lokalen Stand.

REM ---- hoechsten Release-Branch versionssortiert ermitteln ----
set "RELEASE="
for /f "delims=" %%b in ('git for-each-ref --sort^=-v:refname --format^="%%(refname:short)" "%RELEASE_GLOB%"') do (
    if not defined RELEASE set "RELEASE=%%b"
)

if not defined RELEASE (
    echo FEHLER: kein Release-Branch gefunden zu Muster '%RELEASE_GLOB%'.
    exit /b 2
)

echo Hoechster Release-Branch : %RELEASE%
echo Ziel-Branch              : origin/%TARGET%
echo ------------------------------------------------------------

REM ---- pruefen: was fehlt inhaltlich in master? ----
set "MISSING="
for /f "tokens=1,*" %%a in ('git cherry -v "origin/%TARGET%" "%RELEASE%"') do (
    if "%%a"=="+" (
        set "MISSING=1"
        echo   FEHLT  %%b
    )
)

echo ------------------------------------------------------------
if defined MISSING (
    echo ERGEBNIS: NICHT OK - obige Fixes fehlen in %TARGET%.
    set "RC=1"
) else (
    echo ERGEBNIS: OK - %RELEASE% ist vollstaendig in %TARGET% enthalten.
    set "RC=0"
)

REM ---- Bei Doppelklick Fenster offen halten: naechste Zeile einkommentieren ----
REM pause
exit /b %RC%


```` 
