---
title: "Notes Radare2Basics"
layout: post
author: A1phaZer0
category: HackING
---

            +yhy  +hhyyyo          xo          xo
           sMMMMN+dMMMMMs      ydNMMy      sdNMMy
            dNNN  hMMMMMs  ydNMMMMMMy  ydNMMMMMMy
                  hMMMMMsohmNMMMMMMMy+hmNMMMMMMMy
                  yMMMMMs  :/shmNMMMs  :/ohmNMMMy
                  hNNNNNs      :/ohms      :/oydy
                ::///////:         ::          ::
		

<!--more-->
### Command Format ###
```bash
[.][times][cmd][~grep][@[@iter]addr!size][|>pipe]
```  
`.`: Used for interpret something as r2 commands.  
`.r2cmd`: Interpret output of r2cmd as command.  
`. filename`: Interpret r2 script file(`filename`).
`times`: Repeat command specific times.
`~grep`: `~pattern` for pattern, `~[NUM]` for column selection(`~[NUM1-NUM2]`for range), `~:NUM` for row selection.
`cmd @ addr`: Execute `cmd` at the position of address/symbolic name/flag.
`cmd @@`: Execute `cmd` iteratively at different positons.
`cmd @@dbt[abs]`: Execute `cmd` on every backtrace address, sp`s` or bp`b`;
`cmd @@=off1 off2...`: Execute `cmd` on off1, off2, ...
`cmd @@/x 9090`: Execute `cmd` on all search results.
`cmd @@b`: Execute `cmd` on all basic blocks of current function.
`cmd @@i`: Execute `cmd` on all instructions of current function.
`cmd @@f[:fcname]`: Execute `cmd` on all functions or functions matching `fcname`.
`cmd @ !size`: Temporarily change the block size to `size`, so `cmd` will be executed on a different scale. `@` is neccessary even for `cmd` executed at current position.
### Commands ###
**`i`: For information.**
`i\iI`: File info\Binaray info.
`ie\iE\ii`: Entrypoint\Exports\Imports.
`il`: List libraries.
`ir`: List relocs.
`ih`: Show header.
`id`: Debug info.
`ic`: List classes, methods and fields.
`iM`: Show `main` address.
`is`: List symbols.
`iS`: List sections.
`iV`: Version info.
`iz\izz`: Strings in data section\whole file.
`q`: For quiet output, e.g. `isq`.
**`a`: For Analyse.**
`ab`: Analyze the given bytes, output it's meaning.
`af/afr [name] [addr]`: Anaylze function, r means recursively.
`af+ addr name`: Define a function entry.
`afb`: List basic blocks of given function.
`afb+ fcn_addr bblk_addr bblk_size addr_jmp_to false_jmp_to`: Define a basic block at `bblk_addr`, if basic block ends with a jump, then add jump destination, if it's a conditional jump, then also add false branch jump destination.
`afn name addr`: Rename function at `addr` to `name`.
`afl`: List all functions.
`afvr`: List register based args.
`afvb`: List all args/locals(bp based).
`afvb idx name type`: Name ebp-idx to `name`.
`afvs`: List all args/locals(sp based).
`afvs idx name type`: Name esp-idx to `name`.
`afvR name`: List addresses where args/locals accessed as source.
`afvW name`: List addresses where args/locals accessed.
`afvn old_name new_name`: Rename arg/local.
`afvt name new_type`: Change type.
`afv- [name]`: Delete all or specific arg/local.
`agf addr`: Call graph for current function.
`av*`: List virtual functions.
`ar\ar=`: List gpr(general purpose registers) vertically\in columns.
`arr`: Show registers and their reference.
`arc\arc=`: Show conditional flags vertically\in columns.
`ard`: Show registers which value changed.
`aro`: Previous register values.
`art\ar name`: List register types\Show register values of type `name`.
`axF str.|obj.|map.|sym.|loc.|reloc.`: Find reference of flags.
`axf/axt`: Show references from\to here.
**`c`: For Compare.**
`c string`: Compare stuff at current location with string.
`c4\c8 expr`: Compare 4\8 bytes with math expression `expr`.
`cc addr1 @addr2`: Compare hexdump of block at current positon or `addr2` with `addr1`. 
`ccc`: Same as above, just show different lines.
`ccd`: Compare disassembled code.
`cx`: Compare hexpair string.
**`d`: For Debug.**
`db`: List breakpoints.
`db main|addr`: Add a breakpoint at flag or addr.
`db -addr`: Remove a breakpoint.
`dbc main cmd`: Execute `cmd` when hit the breakpoint. `"dbc main cmd1;cmd2"` for multi commands, double quotation marks are necessary.
`dbe\dbd addr`: Enable\disable a breakpoint.
`dc`: Continue.
`dcc\dccu`: Continue until call(step into)\unknown call.
`dcs`: Continue until syscall.
`dcr`: Continue until ret. Step over sub-procedures.
`dcu addr|flag`: Continue until addr.
`dm`: Show memory map.
`dm.`: Show current address memory map.
`do`: Reload.
`doo args`: Reload with args.
`ds num`: Step one instruction or `num` instructions.
`dsf`: Step until end of frame.
`dso num`: Step over `num` instructions.
`dsu addr`: Step until addr.
`dsui\dsuir instr`: Step until instruction `instr`, dsuir for regex.
`dsp`: Skip libs, step into program.
`dx`: Inject opcode.
`dxa asm`: Inject assemble code.
`dxs`: Inject syscall.
**`o`: For Open**
`o`: List opened files.
`o\o+ file`: Open file in read-only\read-write mode.
`oo\oo+\ood`: Reopen current file in read-write\debug mode.
`om`: List all IO maps.
`om.`: Show current IO map.
`omf [mapid] rwx`: Change mapid's rwx flag.
`omfg[+-] rwx`: Change flags of all maps.
**`p`: For Print**
`p6e\p6d len`: Use base64 to encode\decode `len` bytes.
`p8 len`: Show `len` bytes hexpair.
`pa asm`: Show assembled code.
`pd\pD len`: Show `len` disassmebled instructions\Disassemble `len` bytes from current.
`pb\pB len`: Print `len` bits\bytes bitstream.
`px0`: Print hexpair until 0-byte.
`pxb len`: Dump `len` bytes to bits.
`pxd1\pxd2\pxd4 len`: Dump `len` bytes to signed 1\2\4-byte long number. 
`pxf len`: Hexdump of current function.
`pxo len`: Octal dump.
`pxq len`: Quad-word dump.
`pxw len`: Word dump.
`pxx\pxX len`: `len` bytes\words ascii dump. 
`pd N\pd -N`: Print N disassembled instructions forward\backward.
`pdb`: Disassemble basic block.
`pdf`: Disassemble function.
`pdsf`: Print where jumps, calls, xrefs happen.
`pi\pI N`: Print `N` instructions\\`N` bytes.
`pdi N`: Print `N` instructions with offset.
`pfo`: List available binary formats.
`pfo NAME`: Load target binary format.
`pf.elf_header [@ addr]`: Print as elf header.
**`s`: For Seek**
`s\s:`: Print current address(with padded 0s).
`s-\s+`: Undo\redo seek.
`s+ N\s- N`: Seek `N` bytes forward\backward.
`s++ N\s-- N`: Seek `N` block forward\backward.
`sb`: Seek aligned basic block start.
`sf`: Seek to next address after the end of function.
`sg\sG`: Seek to begin\end of section\file.
`sr eip`: Seek to where register points to.
**`S`: For Section**
`S`: List sections.
`S.`: Show current section name.
**`w`: For Write**
`w[1248][+-] N`: Increase\decrease byte\half word\... by N.
`w string`: Write string.
`wa nop | "wa nop;nop"`: Write one instruction\multiple instructions.
`wd addr N`: Write `N` duplicated bytes from `addr` here.
`woe [from to] [step] [wsz]`: Write sequence, wsz is size of each number.
`wopD len`: Wirte De Bruijn pattern of len.
`wopO val`: Find offset of val in De Bruijn sequence.
`wv[1248] val`: Write `val` with size 1\2\4\8 bytes.
`wz string`: Write string with zero terminated.
`wo[asmd] val [@addr [!sz]]`: Write over current data by add\substract\multiply\divide `val`.
`wo[Aox] val [@addr [!sz]]`: Write over current data by add\or\xor `val`.
`wx 9090`: Write hexpair.
**`y`: For Yank**
`y`: List yank buffer information.
`y N [@addr]`: Copy `N` bytes from `addr`.
`yp\yx\ys`: Print contents of clipboard\in hex\as string.
`yy`: Paste clipboard.
**`V`: For Visual**
`c`: Toggle cursor.
`\`: Split mode.
`/`: Search string when cursor on.
`f\f-\F`: Set\remove\browse flag(s).
`p\P`: Panel rotation.
`V`: Function call graph.
`p\P`: Panel rotation in function call graph.
`t\f`: Follow true\false edge.
`g[a-zA-Z*]`: Jump to node labeled as ;[gx].
`x\X`: Jump to xreference\reference.
`hjkl`: Move around.
`HJKL`: Move node.


https://reverser.ninja/2015/23/radare2-cheat-sheet.html
