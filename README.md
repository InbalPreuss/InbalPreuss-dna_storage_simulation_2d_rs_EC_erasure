# Dna Storage

## Usage
### Setting Up the Environment

Before running the simulations, it's recommended to set up a virtual environment to manage dependencies. This ensures that the project's dependencies do not conflict with those of other Python projects on your system.
Create a venv (recommended)
1. Clone the Repository
```console
# Clone main simulation repository
git clone https://github.com/InbalPreuss/dna_storage_simulation_2d_rs_EC_erasure.git
cd dna_storage_simulation_new_error_correction

# Clone RS repository into this directory
git clone https://github.com/InbalPreuss/unireedsolomon.git

```
2. Set Up Python Environment
(a) Install Python 3.10
Ensure you're using Python 3.10, as required by the simulation.

(b) Create and Activate a Virtual Environment
For Unix or MacOS:
```console
python3 -m venv venv
source venv/bin/activate
```

For Windows:
```console
python -m venv venv
.\venv\Scripts\activate
```

(c) Install Dependencies
```console
pip install -r requirements.txt
```
3. Set PYTHONPATH
Set the PYTHONPATH so Python can find your modules correctly:
```console
export PYTHONPATH=$(pwd):$PYTHONPATH
```
On Windows, use:
```console
set PYTHONPATH=%cd%;%PYTHONPATH%
```

4. Run the Simulation Script
The main simulation script is located in the tests directory. To run it:
```console
python3 tests/distributed.py
```
Alternatively, you can run it in the background:
```console
nohup python3 -m tests.distributed &
```
5. Run plots on the data created. The data will be in .
When finished run:
```console
nohup python3 -m dna_storage.plots &
```

### Simulation Description
The results of the simulation runs are summarized in Figure 4 and Figure 5 in "Data Storage Based on Combinatorial Synthesis of DNA Shortmers".
Each run included:
* 30 repeats with random input texts of 10KB each
* |k-mer| = 3
* Ω = {'AAT': 'X1',
       'ACA': 'X2',
       'ATG': 'X3',
       'AGC': 'X4',
       'TAA': 'X5',
       'TCT': 'X6',
       'TTC': 'X7',
       'TGG': 'X8',
       'GAG': 'X9',
       'GCC': 'X10',
       'GTT': 'X11',
       'GGA': 'X12',
       'CAC': 'X13',
       'CCG': 'X14',
       'CTA': 'X15',
       'CGT': 'X16'}
* Barcode length 12nt
* Barcode RS length 4nt
* Payload length 120nt
* Payload RS length 14nt
* Block length 42nt
* block RS length 6nt
* 1000 copies(synthesized) per barcode
* Each run with alphabet of size:
  * |Σ| = 512 (𝑁 = |Ω| = 16, 𝐾 = 3)
  * |Σ| = 4,096 (𝑁 = |Ω| = 16, 𝐾 = 5)
  * |Σ| = 8,192 (𝑁 = |Ω| = 16, 𝐾 = 7)
* Error rates simulated: 0, 0.0001, 0.001, 0.01
* Sampeling rate simulated: 10 – 1000 reads on average per barcode

### Algorithm approce:
1. Encoding  
1.1. Data padding. To fit the binary data onto the molecule, it must be divided by the molecule size and the block size. If the division results in a gap, the data is padded with zeros to close this gap.  
1.2. 2D-error correction using Reed-Solomon decoding. Reed Solomon (RS) is used a total of three times. It is applied lengthwise on each sequence twice, error correcting each barcode sequence and then each payload sequence. It is also used crosswise on all the sequences in one block size.   
2. Synthesis and sequencing.    
2.1. Simulating the synthesis process. The synthesis of each combinatorial sequence was simulated separately.
For a fixed sequence we first draw, from 𝑋`~`𝑁(𝜇 =predtermined, 𝜎^2 = 100), the number of molecules that will represent it. Let this number be x. All k-mers that occur within a single position (cycle) are then generated. To do this, x numbers of the subset are selected, representing the relevant 𝜎. The size of this 
subset is 𝐾, and its members will most likely be represented many times. This random composition is achieved by drawing a total of x independent times, according to 𝑌`~`𝑈(1,𝐾). 1s in 𝜎 are indexed at 1, …, 𝐾, and the appropriate k-mers are “synthesized” in accordance with the drawn index. 
2.2. Mixing. Once all of the molecules are synthesized, they are mixed to mimic real molecules in a container.  
2.3. Error simulation. To replicate a real synthesis and sequencing process, several error scenarios were simulated. These include the three error types Deletion, Insertion, and Substitution of a letter in the sequence, each predefined by an error percentage. A Bernoulli trial is then performed on every sequence and letter position, where 𝑃 is the predefined error, inserting the errors in each position. To replicate the Substitution error, we implemented the error per nucleotide, and for the Insertion and Deletions errors we implemented 
the error on each full k-mer. This method is closest to the expected error scenarios in combinatorial DNA synthesis and sequencing.  
2.4. Reading and sampling. Several different samples were drawn, to analyze their impact on the accuracy of 
the data retrieved.
3. Decoding  
3.1. Sequence retrieval. To retrieve the original sequence, first each sequence barcode undergoes RS error correction. Next, each sequence payload is reviewed individually, and undergoes RS, too. For sequences in the same block, RS is also done, crosswise on the block.  
3.2. Grouping by barcode and determining 𝝈 in each position. Once barcode retrieval is complete, sequences are grouped by the same barcode. In each of the groups, all the sequences are reviewed at the same exact position, where we extract the 𝐾 most common k-mers to determine the 𝜎 in that position. In the process of determining the 𝐾 most common k-mers, we may encounter invalid k-mer (not in Ω). Should an invalid k-mer be encountered in the payload sequence, the following steps are taken:  
• If the length of the sequence is equal to the predetermined length. The sequence is reviewed, and if an invalid k-mer is encountered, which is not part of our alphabet, an 𝑋𝑑𝑢𝑚𝑚𝑦 is inserted instead, followed by skipping 3nts.  
• If the length of the sequence is smaller than the predetermined length. When Δ<SL*, it indicates that there is a deletion in the sequence. We pad it with a dummy nucleotide 𝑅 that restores it to the predetermined length, and then review the sequence.   
• If the length of the sequence is greater than the predetermined length. When Δ>SL*, it indicates that there is an insertion in the sequence. We cut the sequence to restore it to the predetermined length, and then review the sequence.   
* Δ − 𝐶𝑢𝑟𝑟𝑒𝑛𝑡 𝑠𝑒𝑞𝑢𝑒𝑛𝑐𝑒 𝑙𝑒𝑛𝑔𝑡ℎ, 𝑆𝐿 − 𝑃𝑟𝑒𝑑𝑒𝑡𝑒𝑟𝑚𝑖𝑛𝑒𝑑 𝑠𝑒𝑞𝑢𝑒𝑛𝑐𝑒 𝑙𝑒𝑛𝑔𝑡ℎ
Text After 2D Rs Decoding.  
3.3. Missing barcode. After the reading process is complete, if a barcode is found missing, the missing barcode and a dummy sequence is added to enable Reed Solomon to retrieve the data correctly.   
4. Validation  
4.1. Levenshtein distance. After recovering the data, the magnitude of the error in the information recovery process is reviewed, by assessing the Levenshtein distance between the input 𝐼 and output 𝑂.

