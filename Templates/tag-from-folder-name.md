<%*
const pathParts = tp.file.folder(true).split("/");
const outermostFolder = pathParts[0].toLowerCase();
tR += `---\ntags: ${outermostFolder}\n---\n`;
%>