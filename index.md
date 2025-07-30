
{:.no_toc}
* toc
{:toc}


# Abstract
Zero-shot online voice conversion (VC) holds significant promise for real-time communications and entertainment. However, current VC models struggle to preserve semantic fidelity under real-time constraints, deliver natural-sounding conversions, and adapt effectively to unseen speaker characteristics.
To address these challenges, we introduce Conan, a chunkwise online zero-shot voice conversion model that preserves the content of the source while matching the voice timbre and styles of reference speech. Conan comprises three core components: 1) a Stream Content Extractor that leverages Emformer for low-latency streaming content encoding; 2) an Adaptive Style Encoder that extracts fine-grained stylistic features from reference speech for enhanced style adaptation; 3) a Causal Shuffle Vocoder that implements a fully causal HiFiGAN using a pixel-shuffle mechanism. Experimental evaluations demonstrate that Conan outperforms baseline models in both subjective and objective metrics.


# Zero-shot Streaming Voice Conversion
We first show examples of zero-shot streaming voice conversion on VCTK dataset.

### Example 1
<!-- ── begin copy-paste ───────────────────────────────────────────── -->
<table>
  <thead>
    <tr>
      <th style="text-align: center">Source</th>
      <th style="text-align: center">Reference</th>
      <th style="text-align: center">Conan</th>
      <th style="text-align: center">Conan (fastest)</th>
    </tr>
  </thead>

  <tbody>
    <!-- top row: Target + (Ref / Res / Res) for pair #1 -->
    <tr>
      <!-- Target spans all three reference/result rows -->
      <td rowspan="3" style="text-align: center; vertical-align:middle;">
        <audio controls style="width: 150px;">   <!-- 90 % looks nice on wide tables -->
          <source src="wavs/Change_Spks/eg1/p226_191_mic2.wav" type="audio/wav">
        </audio>
      </td>
      <!-- Reference #1 -->
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg1/p243_005_mic2.wav" type="audio/wav">
        </audio>
      </td>
      <!-- Conan result #1 -->
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg1/full/p243_005_mic2_p226_191_mic2.wav" type="audio/wav">
        </audio>
      </td>
      <!-- Other model result #1 -->
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg1/fastest/p243_005_mic2_p226_191_mic2.wav" type="audio/wav">
        </audio>
      </td>
    </tr>
    <!-- pair #2 -->
    <tr>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg1/p249_011_mic2.wav" type="audio/wav">
        </audio>
      </td>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg1/full/p249_011_mic2_p226_191_mic2.wav" type="audio/wav">
        </audio>
      </td>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg1/fastest/p249_011_mic2_p226_191_mic2.wav" type="audio/wav">
        </audio>
      </td>
    </tr>
    <!-- pair #3 -->
    <tr>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg1/p302_031_mic2.wav" type="audio/wav">
        </audio>
      </td>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg1/full/p302_031_mic2_p226_191_mic2.wav" type="audio/wav">
        </audio>
      </td>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg1/fastest/p302_031_mic2_p226_191_mic2.wav" type="audio/wav">
        </audio>
      </td>
    </tr>
  </tbody>
</table>
<!-- ── end copy-paste ─────────────────────────────────────────────── -->

### Example 2
<!-- ── begin copy-paste ───────────────────────────────────────────── -->
<table>
  <thead>
    <tr>
      <th style="text-align: center">Source</th>
      <th style="text-align: center">Reference</th>
      <th style="text-align: center">Conan</th>
      <th style="text-align: center">Conan (fastest)</th>
    </tr>
  </thead>

  <tbody>
    <!-- top row: Target + (Ref / Res / Res) for pair #1 -->
    <tr>
      <!-- Target spans all three reference/result rows -->
      <td rowspan="3" style="text-align: center; vertical-align:middle;">
        <audio controls style="width: 150px;">   <!-- 90 % looks nice on wide tables -->
          <source src="wavs/Change_Spks/eg2/target.wav" type="audio/wav">
        </audio>
      </td>
      <!-- Reference #1 -->
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg2/ref1.wav" type="audio/wav">
        </audio>
      </td>
      <!-- Conan result #1 -->
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg2/full/gen1.wav" type="audio/wav">
        </audio>
      </td>
      <!-- Other model result #1 -->
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg2/fastest/gen1.wav" type="audio/wav">
        </audio>
      </td>
    </tr>
    <!-- pair #2 -->
    <tr>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg2/ref2.wav" type="audio/wav">
        </audio>
      </td>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg2/full/gen2.wav" type="audio/wav">
        </audio>
      </td>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg2/fastest/gen2.wav" type="audio/wav">
        </audio>
      </td>
    </tr>
    <!-- pair #3 -->
    <tr>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg2/ref3.wav" type="audio/wav">
        </audio>
      </td>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg2/full/gen3.wav" type="audio/wav">
        </audio>
      </td>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg2/fastest/gen3.wav" type="audio/wav">
        </audio>
      </td>
    </tr>
  </tbody>
</table>

### Example 3
<!-- ── begin copy-paste ───────────────────────────────────────────── -->
<table>
  <thead>
    <tr>
      <th style="text-align: center">Source</th>
      <th style="text-align: center">Reference</th>
      <th style="text-align: center">Conan</th>
      <th style="text-align: center">Conan (fastest)</th>
    </tr>
  </thead>

  <tbody>
    <!-- top row: Target + (Ref / Res / Res) for pair #1 -->
    <tr>
      <!-- Target spans all three reference/result rows -->
      <td rowspan="3" style="text-align: center; vertical-align:middle;">
        <audio controls style="width: 150px;">   <!-- 90 % looks nice on wide tables -->
          <source src="wavs/Change_Spks/eg3/p231_145_mic2.wav" type="audio/wav">
        </audio>
      </td>
      <!-- Reference #1 -->
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg3/p252_109_mic2.wav" type="audio/wav">
        </audio>
      </td>
      <!-- Conan result #1 -->
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg3/full/p252_109_mic2_p231_145_mic2.wav" type="audio/wav">
        </audio>
      </td>
      <!-- Other model result #1 -->
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg3/full/p252_109_mic2_p231_145_mic2.wav" type="audio/wav">
        </audio>
      </td>
    </tr>
    <!-- pair #2 -->
    <tr>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg3/p276_004_mic2.wav" type="audio/wav">
        </audio>
      </td>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg3/full/p276_004_mic2_p231_145_mic2.wav" type="audio/wav">
        </audio>
      </td>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg3/fastest/p276_004_mic2_p231_145_mic2.wav" type="audio/wav">
        </audio>
      </td>
    </tr>
    <!-- pair #3 -->
    <tr>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg3/p293_034_mic2.wav" type="audio/wav">
        </audio>
      </td>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg3/full/p293_034_mic2_p231_145_mic2.wav" type="audio/wav">
        </audio>
      </td>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg3/fastest/p293_034_mic2_p231_145_mic2.wav" type="audio/wav">
        </audio>
      </td>
    </tr>
  </tbody>
</table>
<!-- ── end copy-paste ─────────────────────────────────────────────── -->

### Example 4
<!-- ── begin copy-paste ───────────────────────────────────────────── -->
<table>
  <thead>
    <tr>
      <th style="text-align: center">Source</th>
      <th style="text-align: center">Reference</th>
      <th style="text-align: center">Conan</th>
      <th style="text-align: center">Conan (fastest)</th>
    </tr>
  </thead>

  <tbody>
    <!-- top row: Target + (Ref / Res / Res) for pair #1 -->
    <tr>
      <!-- Target spans all three reference/result rows -->
      <td rowspan="3" style="text-align: center; vertical-align:middle;">
        <audio controls style="width: 150px;">   <!-- 90 % looks nice on wide tables -->
          <source src="wavs/Change_Spks/eg4/p360_057_mic2.wav" type="audio/wav">
        </audio>
      </td>
      <!-- Reference #1 -->
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg4/p288_002_mic2.wav" type="audio/wav">
        </audio>
      </td>
      <!-- Conan result #1 -->
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg4/full/p288_002_mic2_p360_057_mic2.wav" type="audio/wav">
        </audio>
      </td>
      <!-- Other model result #1 -->
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg4/fastest/p288_002_mic2_p360_057_mic2.wav" type="audio/wav">
        </audio>
      </td>
    </tr>
    <!-- pair #2 -->
    <tr>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg4/p303_025_mic2.wav" type="audio/wav">
        </audio>
      </td>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg4/full/p303_025_mic2_p360_057_mic2.wav" type="audio/wav">
        </audio>
      </td>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg4/fastest/p303_025_mic2_p360_057_mic2.wav" type="audio/wav">
        </audio>
      </td>
    </tr>
    <!-- pair #3 -->
    <tr>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg4/p339_345_mic2.wav" type="audio/wav">
        </audio>
      </td>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg4/full/p339_345_mic2_p360_057_mic2.wav" type="audio/wav">
        </audio>
      </td>
      <td style="text-align: center">
        <audio controls style="width: 150px;">
          <source src="wavs/Change_Spks/eg4/fastest/p339_345_mic2_p360_057_mic2.wav" type="audio/wav">
        </audio>
      </td>
    </tr>
  </tbody>
</table>


# Cross-dataset Comparison
We then demonstrate the cross-dataset performance of our method. During streaming inference, the entire reference speech is first fed into the model to provide timbre and stylistic information. For chunk-wise online inference, the input is processed once it reaches a predefined chunk size before being passed to the model.

In this experiment, speech from VCTK is used as the target speaker, while speech from LibriTTS serves as the source content. Note that StreamVC is not open-sourced and requires f0 as input; hence, it is excluded from this comparison. **All the baseline approaches compared here are offline voice conversion methods**, which are not designed for streaming inference.

### Reference Speaker p231
Reference Speech: <audio controls style="width: 30%; margin: 15px 0;"><source src="wavs/Origins/p231_001_002_003.wav" type="audio/wav"> </audio>

<ruby>Text: He shrugged his shoulders in ungracious acquiescence, while our visitor in hurried words and with much excitable gesticulation poured forth his story.</ruby>
<table>
	<thead>
		<tr>
			<th style="text-align: center">Source</th>
			<th style="text-align: center">Conan</th>
			<th style="text-align: center">Conan (fastest)</th>
			<th style="text-align: center">QuickVC</th>
			<th style="text-align: center">DiffVC</th>
			<th style="text-align: center">VQMIVC</th>
			<th style="text-align: center">PPGVC</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td style="text-align: center"><audio controls style="width: 150px;"><source src="wavs/Origins/Source_Origin_5.wav" type="audio/wav"></audio></td>
			<td style="text-align: center"><audio controls style="width: 150px;"><source src="wavs/p231/Conan/5.wav" type="audio/wav"></audio></td>
			<td style="text-align: center"><audio controls style="width: 150px;"><source src="wavs/p231/Conan_fastest/5.wav" type="audio/wav"></audio></td>
			<td style="text-align: center"><audio controls style="width: 150px;"><source src="wavs/p231/QuickVC/5.wav" type="audio/wav"></audio></td>
			<td style="text-align: center"><audio controls style="width: 150px;"><source src="wavs/p231/DiffVC/5.wav" type="audio/wav"></audio></td>
			<td style="text-align: center"><audio controls style="width: 150px;"><source src="wavs/p231/VQMIVC/5.wav" type="audio/wav"></audio></td>
			<td style="text-align: center"><audio controls style="width: 150px;"><source src="wavs/p231/PPGVC/5.wav" type="audio/wav"></audio></td>
		</tr>
	</tbody>
</table>

### Reference Speaker p360
Reference Speech: <audio controls style="width: 30%; margin: 15px 0;"><source src="wavs/Origins/p360_001_002_003.wav" type="audio/wav"> </audio>

<ruby>Text: He examines the horizon all round with his glass, and folds his arms with the air of an injured man.</ruby>
<table>
	<thead>
		<tr>
			<th style="text-align: center">Source</th>
			<th style="text-align: center">Conan</th>
			<th style="text-align: center">Conan (fastest)</th>
			<th style="text-align: center">QuickVC</th>
			<th style="text-align: center">DiffVC</th>
			<th style="text-align: center">VQMIVC</th>
			<th style="text-align: center">PPGVC</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td style="text-align: center"><audio controls style="width: 150px;"><source src="wavs/Origins/Source_Origin_9.wav" type="audio/wav"></audio></td>
			<td style="text-align: center"><audio controls style="width: 150px;"><source src="wavs/p360/Conan/9.wav" type="audio/wav"></audio></td>
			<td style="text-align: center"><audio controls style="width: 150px;"><source src="wavs/p360/Conan_fastest/9.wav" type="audio/wav"></audio></td>
			<td style="text-align: center"><audio controls style="width: 150px;"><source src="wavs/p360/QuickVC/9.wav" type="audio/wav"></audio></td>
			<td style="text-align: center"><audio controls style="width: 150px;"><source src="wavs/p360/DiffVC/9.wav" type="audio/wav"></audio></td>
			<td style="text-align: center"><audio controls style="width: 150px;"><source src="wavs/p360/VQMIVC/9.wav" type="audio/wav"></audio></td>
			<td style="text-align: center"><audio controls style="width: 150px;"><source src="wavs/p360/PPGVC/9.wav" type="audio/wav"></audio></td>
		</tr>
	</tbody>
</table>
