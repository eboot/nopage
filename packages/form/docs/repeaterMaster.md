# 基本

- layout: default
- order: 0

最简单的用法。

---

## 可执行 DEMO

````js
import Form, { FormItem, FormCore, If, Item } from 'noform';
import repeater  from '../src/repeater';
import * as Antd from 'antd';
import wrapper from '../src/wrapper/antd';
import dialogWrapper from '../src/dialog/antd';
import "./repeater.scss";

const { Button, Input, Radio, Checkbox, Select, InputNumber }  = wrapper(Antd);
const Dialog = dialogWrapper(Antd)
const { TableRepeater, InlineRepeater, Selectify, ActionButton } = repeater({ Dialog, Button, Input, Checkbox, Radio });

const validateConfig = {
    username: { type: 'string', required: true },
};

const publicCountry = [
    { label: 'US', value: 'US' },
    { label: 'GB', value: 'GB' },
    { label: 'AU', value: 'AU' },
    { label: 'CN', value: 'CN' },
    { label: 'BR', value: 'BR' },
    { label: 'EG', value: 'EG' },
    { label: 'DE', value: 'DE' },
];
class Example extends React.Component {
    constructor(props, context) {
        super(props, context);
        this.core = new FormCore({
            values: {
                countryRepeater: [
                    { country: ['US'] },
                    { country: ['AU', 'GB'] }
                ],
                ruleRepeater: [
                    { min: 5, max: 10 },
                    { min: 24, max: 39 },
                    { min: 45, max: 60 },
                ],
                masterRepeater: [
                    // { check: 'bbb'},
                    // { check: 'aaa'}
                    { username: 0 },
                    { username: 0.0 },
                    { username: 1 }
                ],
                int1: 0,
                int2: 1
            },
        });

        window.core = this.core;
        this.formConfig = {
            validateConfig,
            autoValidate: true,
        };
        
        this.publicHandler = {
            afterSetting: (event, repeater) => {
                console.log('event', event, 'repeater', repeater);
                const { formList } = repeater;
                const options = this.getRestCountry(formList);
                formList.forEach((fc) => {
                    fc.setItemProps('country', { options })
                });
                this.core.setItemValue('restCountryDEMO', options);
                this.core.setItemProps('countryRepeater', { hasAdd: (options.length !== 0) });
            }
        };

        this.ruleHandler = {
            afterSetting: (event, repeater) => {
                const { type, index: modifyIndex } = event;
                const { formList } = repeater;
                const values = repeater.getValues();

                console.log('event', event, repeater);

                formList.forEach((ctx, index) => {
                    const validateConfig = this.getValidateConfig(values, index);
                    ctx.setValidateConfig(validateConfig);
                    if (ctx.timer) clearTimeout(ctx.timer);
                    ctx.timer = setTimeout(() => {
                        ctx.validate();    
                    }, 300);
                });
            }
        };
    }

    getValidateConfig = (values, index) => {
        const empty = { type: 'number' };
        const before = values[index - 1];
        const after = values[index + 1];
        const current = values[index];

        const minMin = before && !isNaN(before.max) ? Number(before.max) : null;
        const minMax = current && !isNaN(current.max) ? Number(current.max) : null;

        const maxMin = current && !isNaN(current.min) ? Number(current.min) : null;
        const maxMax = after && !isNaN(after.min) ? Number(after.min) : null;

        console.log(`[${index}]`, [minMin, minMax], [maxMin, maxMax]);

        return {
            min: [
                minMin ? { type: 'number', min: minMin, message: `[${index}]不能小于${minMin}` } : empty,
                minMax ? { type: 'number', max: minMax, message: `[${index}]不能大于${minMax}` } : empty
            ],
            max: [
                maxMin ? { type: 'number', min: maxMin, message: `[${index}]不能小于${maxMin}` } : empty,
                maxMax ? { type: 'number', max: maxMax, message: `[${index}]不能大于${maxMax}` } : empty
            ]
        }
    }

    getRestCountry = (formList) => {
        let restCountry = [...publicCountry];
        formList.forEach(fc => { // 公共池去除本次已选取的
            const countryCode = fc.getItemValue('country') || [];
            restCountry = restCountry.filter(r => countryCode.indexOf(r.value) === -1);
        });

        return restCountry;
    }

    renderTableView = (_, ctx) => {
        const { setLoading, getLoading, getDataSource, getCoreList } = ctx;

        const coreList = getCoreList();
        const dataSource = getDataSource() || [];
        const loading = getLoading();        

        const elements = dataSource.map((item, index) => {            
            const core = coreList[index];
            console.log('coreList', core);

            return <div>
                <div>loading: {loading ? 'on' : 'off'}</div>
                <div>username: {item.username}</div>
                <Button onClick={setLoading.bind(null, true)}>Turn ON Loading</Button>
                <Button onClick={setLoading.bind(null, false)}>Turn OFF Loading</Button>
                <ActionButton core={core} type="delete">删除{index}</ActionButton>
            </div>
        });    

        return <div>
            {elements}
        </div>
    }

    renderTest = () => {
        const repeaterDialogConfig = {
            layout: { label: 6, control: 16 },
            title: 'title123',
            // defaultMinWidth: false,
            // style: { width: '800px' },
            // hasCancel: false,
            // btnAlign: 'left',
        };
        const inlineWrap = { style: { display: 'flex' }};
        const inlinePrefix = { layout: null, inline: true };
        const inlineSuffix = { layout: { label: 6, control: 18 }, inline: true, full: true, style: { flex: 1 }};
        return <FormItem label="masterRepeater" name="masterRepeater" full>
            <TableRepeater
                // view={this.renderTableView}
                dialogConfig={repeaterDialogConfig}
            >
                <div>hello1</div>
                <FormItem name="username" label="username"><Input /></FormItem>
            </TableRepeater>
        </FormItem>
    }

    render() {

        const typeSource = [
            { label: 'aaa', value: 'aaa' },
            { label: 'bbb', value: 'bbb' },
            { label: 'ccc', value: 'ccc' }
        ];

        return (<Form core={this.core} layout={{ label: 6, control: 18 }} defaultMinWidth={false}>
            <div className="example-title">Master Repeater Examples</div>
            {this.renderTest()}
            <FormItem label="int1" name="int1" status="preview" defaultValue="2">
                <Input />
            </FormItem>

            <FormItem label="int2" name="int2" status="preview">
                <Input />
            </FormItem>
            {/* public source */}
            {/* <FormItem label="Public Country Repeater" name="countryRepeater">
                <InlineRepeater asyncHandler={this.publicHandler} multiple>
                    <FormItem label="country" name="country">
                        <Select mode="multiple" options={publicCountry} />
                    </FormItem>
                    <FormItem label="min" name="min"><InputNumber /></FormItem>
                    <FormItem label="abb" name="abb"><InputNumber /></FormItem>
                </InlineRepeater>
            </FormItem> */}
            {/* <FormItem label="Freight Calculator" name="freightCalculator">
                <InlineRepeater multiple>
                    <FormItem label="type" name="type" defaultValue="aaa">
                        <Select options={typeSource} />
                    </FormItem>
                    <FormItem label="rule" multiple>
                        <div>
                            <If when={(values) => values.type === 'aaa'}>
                                <Item name="aaa"><Input placeholder="aaa" /></Item>
                            </If>
                            <If when={(values) => values.type === 'bbb'}>
                                <Item name="bbb"><Input placeholder="bbb" /></Item>
                            </If>
                            <If when={(values) => values.type === 'ccc'}>
                                <Item name="ccc"><Input placeholder="ccc" /></Item>
                            </If>
                        </div>
                    </FormItem>
                </InlineRepeater>
            </FormItem> */}
            {/* <FormItem render={(values) => {
                const availableCountry = (values.restCountryDEMO || []).map(item => item.label).join(', ');
                return <div>Available Country: <span style={{ color: 'red' }}>{availableCountry}</span></div>
            }} />
            
            <FormItem label="Rule Repeater" name="ruleRepeater">
                <InlineRepeater asyncHandler={this.ruleHandler} multiple formConfig={{ autoValidate: true }}>
                    <FormItem label="min" name="min"><InputNumber /></FormItem>
                    <FormItem label="max" name="max"><InputNumber /></FormItem>
                </InlineRepeater>
            </FormItem> */}
        </Form>);
    }
}

ReactDOM.render(<Example />, mountNode);

````

````css
body {
    /* background-color: #000; */
    /* background: #000 !important; */
    /* color: #fff !important; */
}
````