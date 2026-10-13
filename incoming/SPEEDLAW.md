# SpeedLaw

The official name of the get_Still narration speed equation (Voice Calibration SOP v1.0, Oct 13, 2026).

    new_speed = current_speed × (measured_length ÷ target_length)

- Lengths are in seconds, measured with ffprobe on the same calibration text.
- Round to 0.01 and clamp to the ElevenLabs slider range, 0.70 to 1.20.
- Procedure: take 1 at a guess, measure, apply SpeedLaw, take 2.
- Pass: within 0.5 s of the reference voice. Take-to-take variation is about ±1 s.

Examples from the calibration:
- Serene Presence: 1.10 × 14.89 ÷ 16.1 = 1.02, which measured 15.78 s against Christian's 16.12 s.
- Alejo: 0.80 × 14.4 ÷ 17.97 = 0.64, clamped to 0.70, which measured 17.55 s against Jhenny's 17.97 s.
