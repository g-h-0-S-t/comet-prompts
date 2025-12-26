# Comet Prompts

Advanced prompt templates for Perplexity AI's Comet browser automation feature designed to streamline job application workflows through intelligent form filling and email drafting.

## Overview

This repository contains two specialized prompt templates that enable automated job application submissions across multiple platforms. The prompts leverage Comet's browser control capabilities to handle complex application forms, pagination, and email responses while maintaining data accuracy and validation compliance.

**Important**: These are prompt templates with placeholder data (`xxxxxxxxxxxxxxx`). You must replace all placeholders with your actual personal information, resume details, and cover letter text before use.

## Prompts

### 1. **job-list.md** - Multi-Page Job Application Automation
Automates job applications across paginated job listing sites with intelligent skill matching and recursive navigation.

**Key Features:**
- Skill-based job matching (60% threshold for core competencies)
- Smart pagination handling across multiple pages
- Automatic form field population from your provided data
- Validation error resolution
- Progress tracking (applied vs. skipped jobs)
- Rate-limited submissions (2-3 second delays)

**Use Cases:**
- LinkedIn job search results
- Indeed listings
- Naukri.com job boards
- Company career portals with pagination

### 2. **jobs-from-gmail.md** - Gmail-Based Application Agent
Processes job opportunities from Gmail's Updates tab with autonomous pagination and dual-mode application handling.

**Key Features:**
- Gmail pagination control (processes all email pages)
- Dual application modes:
  - **Email Draft Mode**: Generates professional application emails using your cover letter
  - **Form Fill Mode**: Completes ATS and employer portals with your resume data
- Mandatory attachment handling with user interaction protocol
- Continuous operation across unlimited email pages
- No manual confirmation required between applications

**Use Cases:**
- Job alert emails from recruiters
- Direct company outreach emails
- Newsletter job postings
- Automated job board notifications

## Required Information

To use these prompts, you must provide your complete information in the designated data sections. Replace all `xxxxxxxxxxxxxxx` placeholders with:

### Personal Information
- Full legal name and preferred display name
- Date of birth, nationality, gender
- Contact details (phone, email, address)
- LinkedIn profile and GitHub/portfolio URLs
- Work authorization status

### Professional Details
- **Professional Summary**: Your career overview and key achievements
- **Work Experience**: Complete employment history with:
  - Company names, positions, dates (start/end)
  - Detailed responsibilities and accomplishments
  - Technologies and tools used
- **Skills**: Frontend, backend, tools, databases, languages
- **Certifications**: Any relevant professional certifications

### Educational Background
- Degree name, institution, location, dates
- CGPA/percentage/grades
- All levels (Bachelor's, Class 12, Class 10, etc.)

### Additional Data
- **Compensation**: Current and expected CTC/salary
- **Cover Letter**: Full text of your cover letter template
- **Identity Documents**: PAN, Aadhaar, Passport numbers (if required by applications)

## Setup Instructions

### Step 1: Prepare Your Data
1. Open the prompt file you want to use (`job-list.md` or `jobs-from-gmail.md`)
2. Copy the entire content
3. Replace **every** `xxxxxxxxxxxxxxx` placeholder with your actual information
4. For work experience and education sections, add all relevant entries with complete details
5. Insert your full cover letter text in the designated cover letter section

### Step 2: Launch Comet
1. Ensure you have an active Perplexity Pro subscription with Comet access
2. Open Perplexity and navigate to Comet
3. In Comet's browser, log into relevant job sites or Gmail
4. Navigate to the starting page (job listing or Gmail Updates tab)
5. Paste your customized prompt into Comet
6. Comet will take control of the browser and begin automation

### Step 3: Monitor Initial Run
- Watch the first few applications to ensure data accuracy
- Verify that forms are being filled correctly
- Check that your information appears as expected
- Be ready to provide resume/cover letter attachments when prompted

## Advanced Features

### Form Handling
- **Calendar/Date Picker Logic**: Multi-strategy date input (keyboard, DOM manipulation, ARIA controls)
- **Address Parsing**: Smart dropdown matching and fallback to manual entry
- **Widget Interaction**: Supports search-combos, multi-select, radio buttons, checkboxes
- **Validation Resolution**: Automatic error detection and retry mechanisms
- **Date Overlap Handling**: Intelligent adjustment of employment date ranges

### Anti-Detection
- Natural scrolling patterns
- Varied typing speeds
- Realistic mouse movements
- Delay randomization

### Attachment Protocol
1. Comet pauses when mandatory attachments are required
2. User manually attaches resume and cover letter files
3. User types `continue` to resume automation
4. Comet verifies attachment presence and completes submission

## Usage Best Practices

### Before You Start
- **Complete All Placeholders**: Ensure every `xxxxxxxxxxxxxxx` is replaced with real data
- **Verify Accuracy**: Double-check dates, company names, and technical skills
- **Prepare Files**: Have resume and cover letter PDFs ready for manual attachment
- **Review Cover Letter**: Customize the cover letter section for your target roles
- **Test First**: Run on 2-3 applications manually to verify behavior

### During Automation
- Stay nearby to handle attachment requests
- Monitor for CAPTCHA challenges
- Watch for any unusual form behavior
- Be ready to type `continue` when prompted

### After Applications
- Review submitted applications for accuracy
- Keep a log of companies applied to
- Follow up on applications personally
- Update your prompt template if you find errors

## Technical Specifications

### Supported Form Technologies
- Standard HTML forms (`<input>`, `<select>`, `<textarea>`)
- Modern React/Angular/Vue component libraries
- ARIA-compliant accessibility widgets
- Dynamic DOM manipulation (Ajax, SPA frameworks)
- Multi-step application wizards

### Validation Handling
- Required field enforcement
- Format validation (email, phone, dates)
- Range constraints (salary, experience years)
- Custom regex patterns
- Conditional logic dependencies

## Security & Privacy

**Critical Privacy Notes:**
- Your personal data exists only in your customized prompt copy
- No data is stored in this repository or transmitted externally
- Keep your customized prompts private and secure
- Never share your filled-in prompt publicly
- Review each application before final submission
- Consider using a dedicated email for job applications

**Recommended Practices:**
- Store your customized prompt in a secure, private location
- Use password-protected documents if saving locally
- Clear Comet browser cache periodically
- Log out of job sites after automation sessions

## Limitations

- Cannot upload files automatically (requires user intervention)
- CAPTCHA challenges require manual solving
- Some advanced bot detection may block automation
- Portal-specific quirks may require prompt adjustments
- Success rate depends on form complexity and portal compatibility
- User must provide all personal information for automation to work
- Operates exclusively within Comet's browser environment

## Troubleshooting

### Common Issues
- **Forms not filling**: Verify all placeholders are replaced with real data
- **Validation errors**: Check date formats and required field completeness
- **Attachment failures**: Ensure you attach files and type `continue` when prompted
- **Pagination issues**: Allow page loads to complete before interaction
- **Bot detection**: Reduce automation speed or apply manually to some sites
- **Comet timeout**: Break large job lists into smaller batches

## Contributing

Contributions welcome for:
- Additional platform-specific templates
- Improved widget interaction strategies
- Enhanced anti-detection techniques
- Better validation error handling
- Template optimization for specific ATS systems
- Documentation improvements
- Comet-specific optimizations

## License

MIT License - See [LICENSE](LICENSE) file for details

## Disclaimer

**Important Legal and Ethical Notice:**

These prompts are designed to assist with legitimate job applications by automating repetitive form filling using Perplexity AI's Comet browser automation. Users are responsible for:
- Ensuring compliance with website terms of service
- Verifying all submitted information accuracy
- Respecting platform-specific usage policies
- Using automation responsibly and ethically
- Providing truthful and accurate personal information
- Maintaining confidentiality of their customized prompts

**Automated job applications should supplement, not replace, personalized outreach and networking efforts.**

The repository maintainers are not responsible for:
- Misuse of automation tools
- Violations of third-party terms of service
- Inaccurate information submitted through these prompts
- Account suspensions or bans resulting from automation
- Privacy breaches from insecure storage of customized prompts

Use these tools at your own risk and discretion.

---

## Acknowledgments

Built exclusively for [Perplexity AI's Comet](https://www.perplexity.ai/hub/blog/introducing-comet) browser automation platform.

## Support

For issues, questions, or suggestions, please open an issue on the GitHub repository.
