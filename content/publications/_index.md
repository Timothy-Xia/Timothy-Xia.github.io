---
title: 'Publications'
date: 2023-10-24
type: landing

design:
  spacing: '5rem'

# Note: `username` refers to the user's folder name in `content/authors/`

# Page sections
sections:
  - block: collection
    content:
      title: Journal Articles
      text: ''
      filters:
        folders:
          - publications
        publication_type: article-journal
        exclude_featured: false
    design:
      view: citation
  - block: collection
    content:
      title: Conference Papers
      text: ''
      filters:
        folders:
          - publications
        publication_type: paper-conference
        exclude_featured: false
    design:
      view: citation
  # - block: resume-skills
  #   content:
  #     title: Skills & Hobbies
  #     username: me
  # - block: resume-awards
  #   content:
  #     title: Awards
  #     username: me
  # - block: resume-languages
  #   content:
  #     title: Languages
  #     username: me
---
