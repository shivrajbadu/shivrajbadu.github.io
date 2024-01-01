---
layout: post
title:  "Conditional Validations"
date:   2019-01-07 08:01:18 +0545
categories: [Ruby on Rails, Conditional Validations]
tags: [ruby on rails, validation]
---

Senerio:
* User may not provide Name when creating profile, so user name is not compulsory
* But if Name is provided then minimum character should be 3 & max character should be 10

Solutions:

```Ruby
# Normally we do like this and is the best way in this situation
validates :name, length: { minimum: 5, maximum: 15 },
                          allow_blank: true
```

```Ruby
validates :name, length: { minimum: 5, maximum: 15 },
                if: :length_of_name_is_not_zero
```

```Ruby
validates :name, length: { minimum: 5, maximum: 15 },
                unless: Proc.new {|obj| obj.name.length == 0}
```

```Ruby
def length_of_name_is_not_zero
  return false if self.name.length.eql?(0)
  true
end
```

Another example of conditional validation

```Ruby
attr_accessor :stu_field_validate

validates :no_of_students, :presence => { :if => "student_no_validate?" }

def student_no_validate?
 self.stu_field_validate.present? and ['no_of_students'].include?(self.stu_field_validate)
end
```

Pass field value as

```Ruby
<%= f.hidden_field :stu_field_validate, :value => "no_of_students" %>
```

{% include inarticle-adsense.html %}