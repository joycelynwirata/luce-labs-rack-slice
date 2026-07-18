# AI Usage Appendix

## AI Tools Used

- ChatGPT (GPT-5.5)

## How AI Was Used

AI tools were used to:

- Research commercially available sensors, ESP32 development boards, smart plugs, and fans.
- Compare Home Assistant integration options (ESPHome, MQTT Discovery, Matter).
- Draft ESPHome configuration and Home Assistant YAML examples.
- Review documentation structure and improve technical writing.

All product specifications, compatibility, and architecture decisions were manually verified using manufacturer documentation before inclusion.

---

## Example Prompts

### Prompt 1

> Recommend commercially available temperature and humidity sensors compatible with ESPHome and Home Assistant.

### Prompt 2

> Generate an ESPHome configuration for an ESP32 reading an SHT45 over I2C.

### Prompt 3

> Create a Home Assistant automation using VPD to control an exhaust fan with hysteresis.

---

## Example of Incorrect AI Output

An AI-generated recommendation initially suggested a Grove SHT45 module that was not available from the referenced vendor.

This was identified by checking the manufacturer's product catalog, after which the recommendation was replaced with the Adafruit SHT45 breakout and the component comparison was updated.

---

## What I Did Not Delegate

I did not rely on AI to make architectural decisions.

Component selection, buy-versus-build tradeoffs, control strategy, failure mode analysis, and final design choices were reviewed and finalized manually to ensure they aligned with Luce Labs' architectural requirements.

---

## Time Breakdown

| Task | Time |
|------|------|
| Reading project brief | 30 min |
| Product research | 2 hr |
| Architecture design | 1.5 hr |
| Home Assistant configuration | 1.5 hr |
| Documentation | 1 hr |
| Review and refinement | 30 min |

**Total:** Approximately 7 hours
