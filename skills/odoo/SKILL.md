---
name: odoo
description: Use when developing Odoo modules (versions 14.0-19.0, primary target 18.0) - provides module scaffolding, manifest templates, view patterns, security setup, OWL 2.0 frontend development, and debugging guidance
---

# Odoo Development Skill

**IMPORTANT: This skill is automatically loaded for ALL Odoo module development tasks.**

**Skill Source**: https://github.com/atakhadiviom/odoo-skill

A comprehensive Odoo development reference covering module scaffolding, bug fixing, API patterns, and best practices for Odoo 14.0-19.0 (primary: 18.0-19.0).

## Key Discoveries from Production (Odoo 18.0)

### CRITICAL Odoo 18.0 Changes:

1. **View XML must use `<list>` tag** - Odoo 18 renamed `<tree>` to `<list>`
2. **Don't use `<field name="type">` in view records** - type is auto-inferred from arch content
3. **license field is required** (Odoo 17+)
4. **menu ID must include module prefix** - e.g., `module_name.menu_xxx`
5. **access CSV format** - model_id:id format: `model_<module_name>_<model_name>`
6. **tracking=True removed** - In Odoo 18, tracking is no longer a field parameter (use `_track` method instead if needed)
7. **view_mode uses `list` not `tree`** - In act_window, use `view_mode="list,form"` not `view_mode="tree,form"`

## Overview

This skill provides Odoo-specific development guidance including:
- OWL 2.0 frontend framework patterns (Odoo 19.0)
- Python 3.11+ compatibility
- PostgreSQL 15+ considerations
- New field widgets and components
- Enhanced API patterns
- Testing and debugging strategies
- Module scaffolding and specification workflows
- XML-RPC batch import optimization
- Dashboard and graph views
- Scheduled cron jobs

## Odoo Version Quick Reference

| Version | Python | PostgreSQL | LTS | Status |
|---------|--------|------------|-----|--------|
| 14.0 | 3.8 | 12 | No | Maintenance |
| 15.0 | 3.9 | 13 | No | Maintenance |
| 16.0 | 3.10 | 14 | No | Maintenance |
| 17.0 | 3.10 | 14 | Yes | LTS until 2034-05 |
| **18.0** | **3.11** | **15** | No | Stable (Current Target) |
| **19.0** | **3.11+** | **15+** | **Yes** | **Latest LTS** |

## Module Structure

```
module_name/
├── __init__.py
├── __manifest__.py
├── models/
│   ├── __init__.py
│   └── main_model.py
├── views/
│   └── main_model_views.xml
├── security/
│   ├── ir.model.access.csv
│   └── security_rules.xml
├── static/
│   ├── src/
│   │   └── components/
│   │       └── my_component/
│   │           ├── my_component.js
│   │           ├── my_component.xml
│   │           └── my_component.scss
│   └── description/
│       ├── icon.png
│       └── index.html
└── i18n/
    └── module_name.pot
```

## Manifest Template (__manifest__.py)

```python
{
    'name': 'My Odoo Module',
    'version': '18.0.1.0.0',
    'category': 'Quality',
    'summary': 'Brief description',
    'description': """
        Long description
    """,
    'author': 'Guangdong Jufeng',
    'website': '',
    'license': 'LGPL-3',
    'depends': [
        'base',
        'mail',
    ],
    'data': [
        'security/security_rules.xml',
        'security/ir.model.access.csv',
        'views/main_model_views.xml',
        'views/menus.xml',
    ],
    'assets': {
        'web.assets_backend': [
            'module_name/static/src/components/**/*.js',
            'module_name/static/src/**/*.xml',
        ],
    },
    'demo': [
        'demo/demo_data.xml',
    ],
    'installable': True,
    'application': True,
    'auto_install': False,
}
```

## Model Development Pattern

```python
from odoo import models, fields, api, _
from odoo.exceptions import ValidationError, UserError

class MainModel(models.Model):
    _name = 'module_name.main_model'
    _description = 'Main Model Description'
    _inherit = ['mail.thread', 'mail.activity.mixin']
    _order = 'name desc'

    name = fields.Char(
        string='Reference',
        required=True,
        copy=False,
        readonly=True,
        index=True,
        default=lambda self: self._get_default_name()
    )

    state = fields.Selection([
        ('draft', 'Draft'),
        ('confirmed', 'Confirmed'),
        ('done', 'Done'),
        ('cancelled', 'Cancelled'),
    ], string='Status', default='draft', tracking=True)

    partner_id = fields.Many2one(
        'res.partner',
        string='Partner',
        required=True,
        tracking=True,
    )

    line_ids = fields.One2many(
        'module_name.line_model',
        'main_id',
        string='Lines',
    )

    total_amount = fields.Monetary(
        string='Total Amount',
        compute='_compute_total_amount',
        store=True,
        currency_field='currency_id',
    )

    currency_id = fields.Many2one(
        'res.currency',
        string='Currency',
        required=True,
        default=lambda self: self.env.company.currency_id,
    )

    company_id = fields.Many2one(
        'res.company',
        string='Company',
        required=True,
        default=lambda self: self.env.company,
        index=True,
    )

    @api.depends('line_ids.amount')
    def _compute_total_amount(self):
        for record in self:
            record.total_amount = sum(record.line_ids.mapped('amount'))

    @api.constrains('partner_id', 'company_id')
    def _check_partner_company(self):
        for record in self:
            if record.partner_id.company_id and record.partner_id.company_id != record.company_id:
                raise ValidationError(_('Partner must belong to the same company.'))

    @api.model
    def create(self, vals):
        if vals.get('name', _('New')) == _('New'):
            vals['name'] = self.env['ir.sequence'].next_by_code(self._name) or _('New')
        return super().create(vals)

    def action_confirm(self):
        self.write({'state': 'confirmed'})
        return True
```

## View Development (Odoo 18.0)

### Form View

```xml
<record id="view_main_model_form" model="ir.ui.view">
    <field name="name">main.model.form</field>
    <field name="model">module_name.main_model</field>
    <field name="arch" type="xml">
        <form string="Main Model">
            <header>
                <button name="action_confirm" type="object" string="Confirm" 
                        class="btn-primary" invisible="state != 'draft'"/>
                <field name="state" widget="statusbar" 
                       statusbar_visible="draft,confirmed,done"/>
            </header>
            <sheet>
                <div class="oe_title">
                    <label for="name" string="Reference"/>
                    <field name="name" class="oe_inline"/>
                </div>
                <group>
                    <group>
                        <field name="partner_id"/>
                        <field name="date"/>
                    </group>
                    <group>
                        <field name="company_id" groups="base.group_multi_company"/>
                        <field name="total_amount" widget="monetary"/>
                    </group>
                </group>
                <notebook>
                    <page string="Lines" name="lines">
                        <field name="line_ids">
                            <tree editable="bottom">
                                <field name="product_id"/>
                                <field name="quantity"/>
                                <field name="amount" widget="monetary" sum="Total"/>
                            </tree>
                        </field>
                    </page>
                </notebook>
            </sheet>
            <div class="oe_chatter">
                <field name="message_follower_ids"/>
                <field name="message_ids"/>
                <field name="activity_ids"/>
            </div>
        </form>
    </field>
</record>
```

### Menu and Action (IMPORTANT: Use module prefix!)

```xml
<odoo>
    <!-- Define action BEFORE menuitem -->
    <record id="action_main_model" model="ir.actions.act_window">
        <field name="name">Main Model</field>
        <field name="res_model">module_name.main_model</field>
        <field name="view_mode">tree,form</field>
    </record>

    <!-- Menu items - USE module prefix in action! -->
    <menuitem id="menu_root" name="Module" sequence="100"/>
    <menuitem id="menu_main" name="Main" parent="menu_root" 
              action="module_name.action_main_model"/>
</odoo>
```

**CRITICAL: When menuitem references an action, use the FULL external ID with module prefix:**
- Correct: `action="module_name.action_main_model"`
- Wrong: `action="action_main_model"` (will cause "External ID not found" error)

## Security (ir.model.access.csv)

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_main_model_user,main.model.user,model_module_name_main_model,base.group_user,1,1,1,0
access_main_model_manager,main.model.manager,model_module_name_main_model,base.group_system,1,1,1,1
access_line_model_user,line.model.user,model_module_name_line_model,base.group_user,1,1,1,0
```

## OWL 2.0 Frontend (Odoo 19.0)

```javascript
/** @odoo-module **/
import { Component, useState, onWillStart } from "@odoo/owl";
import { useService } from "@web/core/utils/hooks";

export class MyComponent extends Component {
    static template = "module_name.MyComponent";
    static props = {
        record: { type: Object, optional: true },
    };

    setup() {
        this.orm = useService("orm");
        this.notification = useService("notification");
        this.state = useState({ data: [] });
        onWillStart(() => this.loadData());
    }

    async loadData() {
        this.state.data = await this.orm.searchRead(
            this.props.record.resModel,
            [['id', '=', this.props.record.resId]],
            ['name', 'state']
        );
    }
}
```

## Testing

```python
from odoo.tests.common import TransactionCase
from odoo.exceptions import ValidationError

class TestMainModel(TransactionCase):

    def setUp(self):
        super().setUp()
        self.Model = self.env['module_name.main_model']
        self.partner = self.env.ref('base.res_partner_1')

    def test_create_record(self):
        record = self.Model.create({'partner_id': self.partner.id})
        self.assertTrue(record.name)
        self.assertEqual(record.state, 'draft')

    def test_state_transitions(self):
        record = self.Model.create({'partner_id': self.partner.id})
        record.action_confirm()
        self.assertEqual(record.state, 'confirmed')

    def test_constraints(self):
        with self.assertRaises(ValidationError):
            # validation code here
            pass
```

## Common Patterns

### onchange Pattern
```python
@api.onchange('partner_id')
def _onchange_partner_id(self):
    if self.partner_id:
        self.currency_id = self.partner_id.property_purchase_currency_id
```

### Multi-Company Default
```python
company_id = fields.Many2one(
    'res.company',
    string='Company',
    default=lambda self: self.env.company,
)
```

## Debugging Tips

```python
# Add breakpoint
import pdb; pdb.set_trace()

# Logging
import logging
_logger = logging.getLogger(__name__)
_logger.info('Debug: %s', value)
```

## Common Errors and Fixes (Odoo 18.0)

### 1. View type field error and tree tag
**Error:** `Wrong value for ir.ui.view.type: 'tree'`

**Cause:** In Odoo 18:
- DON'T use `<field name="type">tree</field>` in view records
- Use `<list>` tag instead of `<tree>` tag

**Fix:** 
```xml
<!-- WRONG (Odoo 17 and below) -->
<record id="view_xxx" model="ir.ui.view">
    <field name="type">tree</field>
    <field name="arch" type="xml">
        <tree>...</tree>
    </field>
</record>

<!-- CORRECT for Odoo 18 -->
<record id="view_xxx" model="ir.ui.view">
    <field name="arch" type="xml">
        <list>...</list>
    </field>
</record>
```

**IMPORTANT:** Odoo 18 renamed `<tree>` to `<list>` for list views!

### 2. License field required
**Error:** Missing license in manifest

**Cause:** Odoo 17+ requires license field

**Fix:** Always include license:
```python
{
    'license': 'LGPL-3',  # Required in Odoo 17+
}
```

### 3. Menu ID conflicts
**Error:** External ID not found

**Cause:** menuitem action must use full external ID with module prefix

**Fix:** Use module prefix:
```xml
<!-- WRONG -->
<menuitem id="menu_main" action="action_main_model"/>

<!-- CORRECT -->
<menuitem id="module_name.menu_main" action="module_name.action_main_model"/>
```

### 4. Model name in access.csv
**Error:** Permission error

**Cause:** model_id:id must match format `model_<module_name>_<model>`

**Fix:** Use correct pattern:
```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_xxx,xxx,model_module_name_model,base.group_user,1,1,1,0
```

## Migration Notes: 18.0 to 19.0

1. **OWL 2.0**: Convert OWL 1.x to OWL 2.0 syntax
2. **Python 3.11+**: Required
3. **New Widgets**: ai_text_assistant, rich_text_content
4. **Asset Loading**: Use module-based asset loading

## References

- [Odoo Documentation](https://www.odoo.com/documentation/18.0/)
- [Odoo Developer](https://www.odoo.com/documentation/18.0/developer.html)
- [OWL Documentation](https://github.com/odoo/owl)

---

**Skill Source**: https://github.com/atakhadiviom/odoo-skill
**Last Updated**: 2026-04-11
