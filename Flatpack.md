<%*
/* ===================== FLATPACK v.1.1.0 =======================
   Scans your vault's live folder tree and builds ONE clean master doc:
   folders -> headings (by real folder depth), each note's content baked in
   ONCE, repeated embeds turned into "> See:" pointers. No master list to keep.
   Creates a NEW note named <TITLE>, optionally moves it to <FINAL_DESTINATION>,
   and never edits your source notes.

   ---- SETTINGS (edit these) ----------------------------------- */
const FILENAME = "Master";
const TITLE = `${FILENAME}_${tp.date.now("YYYY-MM-DD")}`;
const ROOTS = [];
const EXCLUDE_FOLDERS = ["templates"];
const EXCLUDE_FILES = ["Untitled"];
const OUT_FOLDER = "";
const FINAL_DESTINATION = "";
/* ------------------------------------------------------------- */

const SKIP_GENERATED = /\b(smart[- ]?baked|smart export|baked|unbaked|flatpack)\b/i;
const known = new Set(app.vault.getMarkdownFiles().map(f => f.basename));
const isFolder = c => Array.isArray(c.children);
const levelOf = p => Math.min(6, p.split("/").length + 1);
const baseOf  = r => decodeURIComponent(String(r).split("#")[0].trim()).replace(/^.*\//,"").replace(/\.md$/i,"").trim();

function skipFolder(name){ return EXCLUDE_FOLDERS.includes(name); }
function skipFile(f){ return f.extension!=="md" || f.basename===TITLE || SKIP_GENERATED.test(f.basename) || EXCLUDE_FILES.includes(f.basename); }

function fenceMap(L){
  const BT=String.fromCharCode(96); const c=new Array(L.length).fill(false); let f=null;
  for(let i=0;i<L.length;i++){ const m=L[i].match(new RegExp("^\\s*("+BT+"{3,}|~{3,})"));
    if(m){ if(f===null){f=m[1][0];c[i]=true;} else if(L[i].indexOf(f)>-1){c[i]=true;f=null;} else c[i]=true; } else if(f) c[i]=true; }
  return c;
}

let folders=0, files=0, pointers=0, missing=0;
const EMBED_ALL=/!\[\[([^\]]+)\]\]/g, EMBED_LINE=/^\s*!\[\[([^\]]+)\]\]\s*$/;

async function emitFile(file, out){
  const fileLevel = levelOf(file.path); files++;
  out.push("---"); out.push("");
  out.push("#".repeat(fileLevel)+" "+file.basename); out.push("");
  const L = (await app.vault.read(file)).replace(/\r\n?/g,"\n").split("\n");
  const inCode = fenceMap(L);
  let minLvl=7; for(let i=0;i<L.length;i++){ if(inCode[i])continue; const m=L[i].match(/^(#{1,6})\s+\S/); if(m) minLvl=Math.min(minLvl,m[1].length); }
  const shift = (minLvl<=6) ? (fileLevel+1-minLvl) : 0;
  for(let i=0;i<L.length;i++){
    if(inCode[i]){ out.push(L[i]); continue; }
    const hm=L[i].match(/^(#{1,6})(\s+.*)$/); if(hm){ out.push("#".repeat(Math.min(6,hm[1].length+shift))+hm[2]); continue; }
    const le=L[i].match(EMBED_LINE);
    if(le){ const b=baseOf(le[1]);
      if(known.has(b)){ pointers++; out.push("> 🔁 See: "+b); }
      else if(/\.(xlsx|png|jpe?g|pdf|csv|gif|svg)$/i.test(le[1])){ out.push("> 📎 "+le[1]); }
      else { missing++; out.push("> (unresolved: "+le[1]+")"); }
      continue;
    }
    if(/!\[\[/.test(L[i])){ out.push(L[i].replace(EMBED_ALL,(m,r)=>{ const b=baseOf(r); if(known.has(b)){pointers++;return "(🔁 see "+b+")";} return "("+r+")"; })); continue; }
    out.push(L[i]);
  }
  out.push("");
}

function hasContent(folder){
  for(const c of folder.children){
    if(isFolder(c)){ if(!skipFolder(c.name) && hasContent(c)) return true; }
    else if(!skipFile(c)) return true;
  }
  return false;
}
async function walk(folder, out){
  const kids = folder.children.slice().sort((a,b)=>a.name.localeCompare(b.name,undefined,{numeric:true,sensitivity:'base'}));
  for(const c of kids){
    if(isFolder(c)){
      if(skipFolder(c.name) || !hasContent(c)) continue;
      folders++; out.push(""); out.push("#".repeat(levelOf(c.path))+" "+c.name); out.push("");
      await walk(c, out);
    } else if(!skipFile(c)){
      await emitFile(c, out);
    }
  }
}

function notify(msg){
  try{ if(tp.obsidian && tp.obsidian.Notice){ new tp.obsidian.Notice(msg, 7000); return; } }catch(e){}
  try{ new Notice(msg, 7000); }catch(e){ console.log("[Flatpack]", msg); }
}

try{
  const out = ["# "+TITLE, ""];
  let roots;
  if(ROOTS.length){
    roots = ROOTS.map(r => app.vault.getAbstractFileByPath(r)).filter(f => f && isFolder(f));
  } else {
    roots = [app.vault.getRoot()];
  }
  for(const r of roots){
    if(ROOTS.length){
      if(!skipFolder(r.name) && hasContent(r)){ folders++; out.push(""); out.push("#".repeat(levelOf(r.path))+" "+r.name); out.push(""); await walk(r, out); }
    } else {
      await walk(r, out);
    }
  }
  const baked = out.join("\n").replace(/\n{3,}/g,"\n\n").trim()+"\n";

  const outPath = (OUT_FOLDER ? OUT_FOLDER.replace(/\/$/,"")+"/" : "") + TITLE + ".md";
  const existing = app.vault.getAbstractFileByPath(outPath);
  if(existing) await app.vault.modify(existing, baked);
  else await app.vault.create(outPath, baked);

  let finalPath = outPath;
  if(FINAL_DESTINATION){
    finalPath = FINAL_DESTINATION.replace(/\/$/,"") + "/" + TITLE + ".md";
    try { await app.vault.adapter.move(outPath, finalPath); } catch(e) { notify("Move failed: " + e.message); }
  }

  notify("Flatpack: " + files + " files, " + folders + " folders, " + pointers + " repeats packed away" + (missing? ", " + missing + " unresolved links flagged" : "") + ". Saved to " + (FINAL_DESTINATION || "vault root") + ".");
  try{ const f = app.vault.getAbstractFileByPath(finalPath); if(f) { await app.workspace.getLeaf(false).openFile(f); if(navigator.share) navigator.share({title: TITLE, text: TITLE + ".md"}); } }catch(_){}
}catch(e){
  notify("Flatpack error: " + e.message);
}
%>
