import React, { Component } from "react";
import {
  Row,
  Col,
  Tabs,
  Icon,
  Form,
  Input,
  Select,
  Radio,
  Button,
  Badge
} from "antd";
import AppcodeAC from "../../../Common/AppcodeAC";
import { AppcodeContext } from "../../../context";
import { ALIAS_VS } from "../../../../global";

const TabPane = Tabs.TabPane;
const FormItem = Form.Item;
const Option = Select.Option;
const RadioGroup = Radio.Group;

const formItemLayout = {
  labelCol: {
    md: { span: 24 },
    lg: { span: 8 },
    xl: { span: 6 }
  },
  wrapperCol: {
    md: { span: 24 },
    lg: { span: 16 },
    xl: { span: 14 }
  }
};

const formItemLayoutWithOutLabel = {
  wrapperCol: {
    md: { span: 24, offset: 0 },
    lg: { span: 14, offset: 8 },
    xl: { span: 14, offset: 6 }
  }
};

class ServerNewForm extends Component {
  state = {
    data: this.props.data,
    errors: this.props.errors,
    warns: this.props.warns,
    appcode: this.props.data.appcode.slice()
  };
  onChange = (
    event,
    type,
    type_key,
    field,
    field_key = null,
    subfield = null
  ) => {
    const value = event.target.value;
    this.props.onChange(value, type, type_key, field, field_key, subfield);
  };
  onSelect = (
    event,
    type,
    type_key,
    field,
    field_key = null,
    subfield = null
  ) => {
    const value = event;
    this.props.onChange(value, type, type_key, field, field_key, subfield);
  };
  renderGTM = (key, gtm) => {
    const err = this.props.errors.gtm[key];
    return (
      <TabPane
        tab={
          <span>
            <Icon type="global" />
            {gtm.name}
          </span>
        }
        key={key}
      >
        <div style={{ margin: 40 }} />
        <Form>
          <FormItem {...formItemLayout} label="FQDN">
            <Col span={16}>
              <FormItem
                hasFeedback
                validateStatus={err.wideip && "error"}
                help={err.wideip}
              >
                <Input
                  placeholder="x01gnwatapp1v"
                  value={gtm.wideip}
                  onChange={e => this.onChange(e, "gtm", key, "wideip")}
                />
              </FormItem>
            </Col>
            <Col span={8}>
              <span
                style={{
                  display: "inline-block",
                  width: "100%",
                  textAlign: "left",
                  paddingLeft: 5
                }}
              >
                .gtment.dbs.com
              </span>
            </Col>
          </FormItem>
        </Form>
      </TabPane>
    );
  };
  countError = err => {
    // count all other errors
    let num = 0;
    num += err.vip ? 1 : 0;
    num += err.vipport ? 1 : 0;
    num += err.snatip ? 1 : 0;
    num += err.active ? 1 : 0;
    // count server errors
    Object.values(err.servers).forEach(item => {
      num += Object.keys(item).filter(attr => item[attr]).length;
    });
    return num;
  };
  renderLTM = (key, ltm) => {
    const totalservers = Object.keys(ltm.servers).length;
    const serverList = Object.keys(ltm.servers).sort(
      (a, b) => parseInt(a, 10) - parseInt(b, 10)
    );
    const serverForms = serverList.map((svr_key, num) =>
      this.renderServer(svr_key, num, key, totalservers)
    );
    const err = this.props.errors.ltm[key];
    const tab_err = this.countError(err);
    return (
      <TabPane
        tab={
          <Badge count={tab_err} offset={[0, 5]}>
            <span>
              <Icon type="fork" />
              {`${ALIAS_VS} (${ltm.vip}:${ltm.vipport})`}
            </span>
          </Badge>
        }
        key={key}
      >
        <Form>
          <FormItem
            {...formItemLayout}
            style={{ margin: 0 }}
            label="Virtual IP"
          >
            <Col span={16}>
              <FormItem
                hasFeedback
                validateStatus={err.vip && "error"}
                help={err.vip}
              >
                <Input
                  placeholder="10.64.1.1"
                  value={ltm.vip}
                  disabled={true}
                />
              </FormItem>
            </Col>
            <Col span={1}>
              <span
                style={{
                  display: "inline-block",
                  width: "100%",
                  textAlign: "center"
                }}
              >
                :
              </span>
            </Col>
            <Col span={7}>
              <FormItem
                hasFeedback
                validateStatus={err.vipport && "error"}
                help={err.vipport}
              >
                <Input placeholder="443" value={ltm.vipport} disabled={true} />
              </FormItem>
            </Col>
          </FormItem>
          <FormItem
            {...formItemLayout}
            label="SNAT IP"
            hasFeedback
            validateStatus={err.snatip && "error"}
            help={err.snatip}
          >
            <Input
              placeholder="10.64.200.1"
              value={ltm.snatip}
              disabled={true}
            />
          </FormItem>
          <FormItem
            {...formItemLayout}
            label="Servers State"
            hasFeedback
            validateStatus={err.active && "error"}
            help={err.active}
          >
            <RadioGroup
              value={ltm.active}
              onChange={e => this.onChange(e, "ltm", key, "active")}
            >
              <Radio value="No">Active-Active</Radio>
              <Radio value="Yes">Active-Passive</Radio>
            </RadioGroup>
          </FormItem>
          {serverForms}
          <FormItem {...formItemLayoutWithOutLabel}>
            <Button
              type="dashed"
              onClick={this.props.addServer}
              style={{ width: "87.5%" }}
              id={key}
            >
              <Icon type="plus" /> Add Server
            </Button>
          </FormItem>
        </Form>
      </TabPane>
    );
  };
  renderServer = (svr_key, index, ltm_key, total) => {
    const svr = this.props.data.ltm[ltm_key].servers[svr_key];
    const err = this.props.errors.ltm[ltm_key].servers[svr_key];
    const active = this.props.data.ltm[ltm_key].active;
    return (
      <FormItem
        {...(index === 0 ? formItemLayout : formItemLayoutWithOutLabel)}
        label={index === 0 && "Servers"}
        style={{ margin: 0 }}
        key={svr_key}
      >
        <Row
          style={
            err.ipaddress || err.port
              ? { width: "100%", height: "59px" }
              : { width: "100%" }
          }
        >
          <Col span={14}>
            <FormItem
              hasFeedback
              validateStatus={err.ipaddress && "error"}
              help={err.ipaddress}
              style={{ margin: 0 }}
            >
              <Input
                placeholder="10.64.1.1"
                value={svr.ipaddress}
                onChange={e =>
                  this.onChange(
                    e,
                    "ltm",
                    ltm_key,
                    "servers",
                    svr_key,
                    "ipaddress"
                  )
                }
              />
            </FormItem>
          </Col>
          <Col span={1}>
            <span
              style={{
                display: "inline-block",
                width: "100%",
                textAlign: "center"
              }}
            >
              :
            </span>
          </Col>
          <Col span={6}>
            <FormItem
              hasFeedback
              validateStatus={err.port && "error"}
              help={err.port}
              style={{ margin: 0 }}
            >
              <Input
                placeholder="443"
                value={svr.port}
                onChange={e =>
                  this.onChange(e, "ltm", ltm_key, "servers", svr_key, "port")
                }
              />
            </FormItem>
          </Col>
          {total > 1 ? (
            <Col span={3}>
              <Icon
                className="dynamic-delete-button"
                type="minus-circle-o"
                disabled={total.length === 1}
                onClick={() => this.props.removeServer(ltm_key, svr_key)}
              />
            </Col>
          ) : null}
        </Row>
        <Row style={{ width: "100%" }}>
          <Col span={14}>
            <FormItem
              hasFeedback
              validateStatus={err.hostname && "error"}
              help={err.hostname}
            >
              <Input
                placeholder="x01gcldtestapp1a"
                value={svr.hostname}
                onChange={e =>
                  this.onChange(
                    e,
                    "ltm",
                    ltm_key,
                    "servers",
                    svr_key,
                    "hostname"
                  )
                }
              />
            </FormItem>
          </Col>
          <Col span={1}>
            <span
              style={{
                display: "inline-block",
                width: "100%",
                textAlign: "center"
              }}
            />
          </Col>
          <Col span={6}>
            <FormItem
              hasFeedback
              validateStatus={err.active && "error"}
              help={err.active}
            >
              <Select
                value={svr.active}
                disabled={active === "No"}
                onChange={e =>
                  this.onSelect(e, "ltm", ltm_key, "servers", svr_key, "active")
                }
              >
                <Option value="Yes">Active</Option>
                <Option value="No">Passive</Option>
              </Select>
            </FormItem>
          </Col>
        </Row>
      </FormItem>
    );
  };
  tabbasicError = (errors, ltm_err) => {
    let err_count = 0;
    if (errors.appcode) err_count += 1;
    if (ltm_err.vs_name) err_count += 1;
    if (ltm_err.uat_virtual) err_count += 1;
    return err_count;
  };
  getOneError = (ltms_err, attr) => {
    const errors = Object.values(ltms_err)
      .map(err => (attr in err ? err[attr] : null))
      .filter(err => err != null);
    if (errors.length > 1) return errors[0];
    else if (errors) return errors[0];
    else return "";
  };
  render() {
    const { data, errors, elastic } = this.props;
    const ltm = Object.entries(this.props.data.ltm)[0];
    const ltm_err = this.props.errors.ltm[ltm[0]];
    const uat_vs_err = this.getOneError(this.props.errors.ltm, "uat_virtual");
    const tab1_err = this.tabbasicError(errors, ltm_err, uat_vs_err);
    return (
      <div>
        <Tabs defaultActiveKey={ltm[0]}>
          <TabPane
            tab={
              <Badge count={tab1_err} offset={[0, 5]}>
                <span>
                  <Icon type="info-circle-o" />
                  Basic
                </span>
              </Badge>
            }
            key="basic"
          >
            <div style={{ marginBottom: 20, textAlign: "center" }}>
              <i>
                Click on <Icon type="info-circle-o" /> to show more information
                on the option.
              </i>
            </div>
            <Form>
              <FormItem
                {...formItemLayout}
                label={ALIAS_VS}
                hasFeedback
                validateStatus={ltm_err.vs_name && "error"}
                help={ltm_err.vs_name}
              >
                <Input
                  placeholder="W01GO365ADFS1Z"
                  value={ltm[1].vs_name}
                  disabled={true}
                  onChange={e => this.onChange(e, "ltm", ltm[0], "vs_name")}
                />
              </FormItem>
              <FormItem
                {...formItemLayout}
                label="Appcode"
                hasFeedback
                validateStatus={errors.appcode && "error"}
                help={errors.appcode}
              >
                <AppcodeContext.Consumer>
                  {ac_list => (
                    <AppcodeAC
                      app_list={ac_list}
                      appcode={data.appcode}
                      onSelect={this.onSelect}
                      disabled={this.state.appcode ? true : false}
                    />
                  )}
                </AppcodeContext.Consumer>
              </FormItem>
              {elastic === 1 && (
                <FormItem
                  {...formItemLayout}
                  label="UAT VIP"
                  hasFeedback
                  validateStatus={uat_vs_err && "error"}
                  help={uat_vs_err}
                >
                  <Input
                    placeholder="10.92.228.1"
                    value={ltm[1].uat_virtual}
                    onChange={e =>
                      this.onChange(e, "ltm", ltm[0], "uat_virtual")
                    }
                  />
                </FormItem>
              )}
            </Form>
          </TabPane>
          {Object.entries(this.props.data.ltm).map(([key, value]) =>
            this.renderLTM(key, value)
          )}
        </Tabs>
      </div>
    );
  }
}

export default ServerNewForm;
