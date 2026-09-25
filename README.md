# Fragments, repeats, and assembly

Open `index.html` in a browser. The simulator runs offline without dependencies or installation.

## Student workflow

1. Set total genome length (200–2,000 bp), repeat length, and copy number; inspect a small linear genome.
2. Set full-length fragment/read size and target depth (default 3×).
3. Inspect sampled fragments and their DNA sequences.
4. Step through actual sequence overlap joins.
5. Inspect contigs and their exact placements against the original genome.

Changing fragment length, depth, or minimum overlap preserves the reference genome. Changing repeat settings preserves the selected genome length. With zero repeat copies, repeat-length changes also preserve the DNA sequence. Repeat-copy choices are limited to what fits, allowing at least 20 bp in each background block. The Gaps row shows uncovered bases separately from repeat regions, and lists their exact coordinates. Sampling and genome buttons advance displayed deterministic seeds. Presets provide short reads, long reads on the same repeated genome, and a no-repeat example. A FASTA-style text download includes parameters, seeds, reference, reads, contigs, and true/exact coordinates.

## Scientific scope

One error-free, linear, forward-oriented genome of the selected total length. Remaining background bases are distributed as evenly as possible among blocks alternating with identical direct repeat copies. Genome length stays fixed with repeat settings; zero copies leaves the entire genome as single-copy genomic DNA. Each sampled fragment is sequenced in full; no paired ends or sequencing error. Valid starts are sampled uniformly with replacement. Read length is capped at genome length; read count is ceil(target depth × genome length / effective read length). Depth is total sequenced bases / genome length, not fraction covered. Linear ends have lower expected coverage.

The assembler receives only sequences. Duplicate/contained strings are removed. Exact suffix–prefix overlaps meeting the threshold are considered. The longest overlap that is the unique best candidate at both ends is merged; pruning and overlap evaluation repeat. Ambiguous equal-best joins are left unresolved. This heuristic can still misassemble or collapse repeats. It is a teaching model, not an optimal or production assembler.

Whole contigs are searched exactly against the reference after assembly. Multiple placements are displayed separately. Red contigs have no end-to-end exact match; their bar starts are placeholders, not alignments. Union coverage of all matching placements can overstate copy-number recovery. Complete reconstruction requires exactly one contig equal to the reference. A longer read at the same nominal depth is not guaranteed to give better coverage in one random draw.

## Validation

`node test.cjs` checks 200 parameter/seed/size scenarios, known joins, tied overlaps, containment, duplicated matches, coverage totals, read retention, exact placement correctness, and full-genome reads. Additional checks cover repeat-fit constraints, repeat-length independence with zero copies, and 600-read assembly at the maximum genome length and depth.

`node browser-test.cjs` uses the workspace Playwright installation and locally installed Chrome to exercise all stages, controls, presets, sequence inspection, export, genome-size changes, repeat limits, zero-repeat gap labels, and 360-pixel layout. Browser launch may require permission outside a restricted execution sandbox. No Canvas or hosted deployment is included.

## Assembly comparison and repeat-only reads

Assembly now shows the same final contig placements and reference-coverage gaps as Compare, plus exact reference intervals for the left, right, and joined sequences at each join. Final placements remain fixed while individual joins are inspected. Diagnostic text distinguishes zero coverage, multiple placements, and failed end-to-end matches, and explains why coverage alone does not prove a unique assembly.

Purple marks reads whose entire true source interval lies within one inserted repeat. Read selection labels and sequence color carry the same flag; original repeat-only reads are also purple if they participate directly in a displayed join. The download includes `repeat_only`. Flags are teaching annotations from the reference, never assembler inputs. All reads remain in coverage and assembly; repeat-only reads lack unique flanks and cannot distinguish identical copies, but can contribute sequence evidence.

## Reverse-greedy strategy

Choose **Reverse greedy · longest extension** in the Assembly strategy control. Settings and sampled reads are preserved when switching strategies. After deduplicating and removing contained reads, this mode seeds the pair producing the longest merged sequence among pairs with qualifying exact overlaps, then extends either end with the read adding the most bases. For a given pair, the longest exact overlap is used. Ties prefer greater overlap, then stable read order. Reads contained in the growing contig are omitted from the construction path. When extension stops, another contig is started from remaining reads.

Assembly lists the selected read IDs for each final contig. Each displayed join includes a scaled diagram of the selected reads at their positions within that assembled sequence, plus the number of bases added. Purple retains the repeat-only flag. Reference comparisons remain separate and can expose incorrect joins. This is a heuristic for a short construction path, not a globally minimum read cover, and longest extensions need not be the most reliable joins. It uses no true reference positions to choose reads. Tests cover the longest-pair seed objective and 24 parameter combinations with every path position verified against the actual read sequence.
