import React, { Component } from "react";
import { Row, Col, Card, Icon, Popover, Table, Button, Tree } from "antd";
import { ALIAS_VS } from "../../../../global";

const TreeNode = Tree.TreeNode;

// const gridStyle1 = {
//   width: '100%',
//   padding: '0.5rem',
//   textAlign: 'center',
//   backgroundColor: '#d9f7be'
// };

// const gridStyle2 = {
//   width: '100%',
//   padding: '0.5rem',
//   textAlign: 'center',
//   backgroundColor: '#e8e8e8'
// };

const gridStyle1 = {
  color: "#237804"
};

const gridStyle2 = {
  color: "#8c8c8c"
};

class NewDiagram extends Component {
  state = {
    tree_select: [
      Object.keys(this.props.data.ltm).sort(
        (a, b) => parseInt(a, 10) - parseInt(b, 10)
      )[0]
    ]
  };
  renderVIP = ltm => {
    const data = (
      <span>
        <span>
          <b>VIP: </b>
          {ltm.vip || ltm.vipport
            ? `${ltm.vip}:${ltm.vipport}`
            : "<Pending VIP>"}
        </span>
        <br />
        <span>
          <b> SNAT: </b>
          {ltm.snatip ? ltm.snatip : "<Pending SNAT>"}
        </span>
      </span>
    );
    return data;
  };
  renderSNAT = ltm => {
    return (
      <span>
        <b>SNAT: </b>
        {ltm.snatip ? ltm.snatip : "<Pending SNAT>"}
      </span>
    );
  };
  renderServer = (active, server) => {
    return (
      <span
        style={
          active === "Yes" && server.active === "No" ? gridStyle2 : gridStyle1
        }
      >
        <b>Server: </b>
        {server.ipaddress || server.port
          ? `${server.ipaddress}:${server.port}`
          : "<Pending Server>"}
      </span>
    );
  };
  renderDetail = tree_select => {
    const treetype_array = tree_select[0].match(/[\d]+/g);

    if (treetype_array.length === 1) {
      return this.renderVIPDetail(treetype_array);
    } else {
      return this.renderServerDetail(treetype_array);
    }
  };
  renderServerDetail = treetype_array => {
    const ltm = this.props.data.ltm[treetype_array[0]];
    const server = ltm.servers[treetype_array[1]];
    const columns = [
      {
        title: "Configuration",
        dataIndex: "config",
        key: "config"
      },
      {
        title: "Value",
        dataIndex: "value",
        key: "value"
      }
    ];
    const tablecontent = [
      {
        config: "Hostname",
        value: server.hostname
      },
      {
        config: "IP Address",
        value: server.ipaddress
      },
      {
        config: "Port",
        value: server.port
      },
      {
        config: "Active",
        value: `${
          server.active === "No" && ltm.active === "Yes" ? "No" : "Yes"
        }`
      }
    ];
    const content = (
      <Table
        columns={columns}
        dataSource={tablecontent}
        pagination={false}
        rowKey="config"
        size="small"
      />
    );
    return content;
  };
  renderVIPDetail = treetype_array => {
    const ltm = this.props.data.ltm[treetype_array[0]];
    const cs = ltm.clientssl;
    const ssl_content = (
      <React.Fragment>
        <div>
          <Icon type="man" className={cs.name ? "green-icon" : "red-icon"} />
          <b>Name:</b> {cs.name}
        </div>
        {ltm.app_ssl === "Yes" && (
          <div>
            <Icon
              type="idcard"
              className={cs.p12 ? "green-icon" : "red-icon"}
            />
            <b>File:</b> {cs.p12 && cs.p12.name}
          </div>
        )}
        {ltm.app_ssl === "Exist" && (
          <React.Fragment>
            <div>
              <Icon
                type="idcard"
                className={cs.cert ? "green-icon" : "red-icon"}
              />
              <b>Cert:</b> {cs.cert && cs.cert.name}
            </div>
            <div>
              <Icon type="key" className={cs.key ? "green-icon" : "red-icon"} />
              <b>Key:</b> {cs.key && cs.key.name}
            </div>
            <div>
              <Icon type="disconnect" className="green-icon" />
              <b>Chain:</b> {cs.chain && cs.chain.name}
            </div>
          </React.Fragment>
        )}
      </React.Fragment>
    );
    const columns = [
      {
        title: "Configuration",
        dataIndex: "config",
        key: "config"
      },
      {
        title: "Value",
        dataIndex: "value",
        key: "value"
      }
    ];
    const tablecontent = [
      {
        config: "Application Idle Timeout",
        value: ltm.timeout
      },
      {
        config: "Embed Source IP in HTTP Header",
        value: ltm.xforward
      },
      {
        config: "Allow session to persist on same server via",
        value: ltm.sticky
      },
      {
        config: "Require SSL from Client to Loadbalancer",
        value: ltm.app_ssl !== "No" ? "Yes" : "No"
      },
      {
        config: "Require SSL from Loadbalancer to your app",
        value: ltm.servers_ssl
      }
    ];
    const content = (
      <Table
        columns={columns}
        dataSource={tablecontent}
        pagination={false}
        rowKey="config"
        size="small"
      />
    );
    return (
      <div>
        <div style={{ marginBottom: "1rem" }}>
          Name: {ltm.vs_name ? ltm.vs_name : ltm.name}
          {ltm.app_ssl === "Yes" || ltm.app_ssl === "Exist" ? (
            <div style={{ float: "right" }}>
              <Popover
                content={ssl_content}
                title="ClientSSL"
                placement="right"
              >
                <Button
                  type={
                    ltm.app_ssl === "Exist"
                      ? cs.name && cs.cert && cs.key
                        ? "primary"
                        : "danger"
                      : cs.name && cs.p12
                      ? "primary"
                      : "danger"
                  }
                >
                  ClientSSL
                </Button>
              </Popover>
            </div>
          ) : null}
          <br />
          Loadbalancing: {ltm.loadbalance}
        </div>
        <div>{content}</div>
      </div>
    );
  };
  renderSLB = () => {
    if (this.props.data.ltm) {
      const span_num = Object.keys(this.props.data.ltm).length > 1 ? 12 : 24;

      return (
        <Row
          type="flex"
          justify="center"
          style={{ textAlign: "center" }}
          gutter={16}
        >
          {Object.entries(this.props.data.ltm).map(([key, value]) => {
            const cs = value.clientssl;
            const ssl_content = (
              <React.Fragment>
                <div>
                  <Icon
                    type="man"
                    className={cs.name ? "green-icon" : "red-icon"}
                  />
                  <b>Name:</b> {cs.name}
                </div>
                <div>
                  <Icon
                    type="idcard"
                    className={cs.cert ? "green-icon" : "red-icon"}
                  />
                  <b>Cert:</b> {cs.cert && cs.cert.name}
                </div>
                <div>
                  <Icon
                    type="key"
                    className={cs.key ? "green-icon" : "red-icon"}
                  />
                  <b>Key:</b> {cs.key && cs.key.name}
                </div>
                <div>
                  <Icon
                    type="disconnect"
                    className={cs.chain ? "green-icon" : "red-icon"}
                  />
                  <b>Chain:</b> {cs.chain && cs.chain.name}
                </div>
              </React.Fragment>
            );
            return (
              <Col span={span_num} key={key}>
                <Card>
                  <b>{ALIAS_VS}</b>
                  <br />
                  {value.vs_name ? value.vs_name : value.name}
                  {value.app_ssl === "Yes" || value.app_ssl === "Exist" ? (
                    <Popover
                      content={ssl_content}
                      title="ClientSSL"
                      placement="bottom"
                    >
                      <div>
                        <br />
                        <Button
                          type={
                            cs.name && cs.cert && cs.key && cs.chain
                              ? "primary"
                              : "danger"
                          }
                        >
                          ClientSSL
                        </Button>
                      </div>
                    </Popover>
                  ) : null}
                </Card>
              </Col>
            );
          })}
        </Row>
      );
    }
    return null;
  };
  renderLBM = () => {
    if (this.props.data.ltm) {
      const span_num = Object.keys(this.props.data.ltm).length > 1 ? 12 : 24;
      return (
        <Row
          type="flex"
          justify="center"
          style={{ textAlign: "center" }}
          gutter={16}
        >
          {Object.entries(this.props.data.ltm).map(([key, value]) => {
            const columns = [
              {
                title: "Configuration",
                dataIndex: "config",
                key: "config"
              },
              {
                title: "Value",
                dataIndex: "value",
                key: "value"
              }
            ];
            const tablecontent = [
              {
                config: "Application Idle Timeout",
                value: value.timeout
              },
              {
                config: "Embed Source IP in HTTP Header",
                value: value.xforward
              },
              {
                config: "Allow session to persist on same server via",
                value: value.sticky
              },
              {
                config: "Require SSL from Client to Loadbalancer",
                value: value.app_ssl
              },
              {
                config: "Require SSL from Loadbalancer to your app",
                value: value.servers_ssl
              }
            ];

            const content = (
              <Table
                columns={columns}
                dataSource={tablecontent}
                pagination={false}
                rowKey="config"
              />
            );
            return (
              <Col span={span_num} className="column-mb-h" key={key}>
                <Popover
                  placement="right"
                  title={`${value.vs_name ? value.vs_name : value.name} Config`}
                  content={content}
                >
                  <Card
                    bordered={false}
                    hoverable
                    bodyStyle={{ padding: "0.8rem" }}
                  >
                    <div>{value.loadbalance}</div>
                    <div>
                      <Icon type="fork" className="lb-icon-flip" />
                    </div>
                    <div>
                      <small>
                        <i>Hover for more info</i>
                      </small>
                    </div>
                  </Card>
                </Popover>
              </Col>
            );
          })}
        </Row>
      );
    }
    return null;
  };
  renderSVR = () => {
    if (this.props.data.ltm) {
      const span_num = Object.keys(this.props.data.ltm).length > 1 ? 12 : 24;
      return (
        <Row
          type="flex"
          justify="center"
          style={{ textAlign: "center" }}
          gutter={16}
        >
          {Object.entries(this.props.data.ltm).map(([key, value]) => {
            return (
              <Col span={span_num} key={key}>
                <Card title="Servers" bordered={false}>
                  {Object.entries(value.servers).map(([skey, sval]) => {
                    if (sval.ipaddress || sval.port) {
                      return (
                        <Card.Grid
                          key={skey}
                          style={
                            value.active === "Yes" && sval.active === "No"
                              ? gridStyle2
                              : gridStyle1
                          }
                        >
                          {sval.ipaddress || sval.port
                            ? `${sval.ipaddress}:${sval.port}`
                            : "Host"}
                        </Card.Grid>
                      );
                    } else {
                      return null;
                    }
                  })}
                </Card>
              </Col>
            );
          })}
        </Row>
      );
    }
    return null;
  };
  render() {
    const { data } = this.props;
    const { tree_select } = this.state;
    return (
      <div>
        {data.ltm ? (
          <Row gutter={16}>
            <Col xs={{ span: 24 }} sm={{ span: 8 }}>
              <Tree
                showLine
                defaultExpandAll
                defaultSelectedKeys={tree_select}
                onSelect={(selectedKeys, info) =>
                  this.setState({ tree_select: selectedKeys })
                }
              >
                {Object.entries(data.ltm).map(([key, value]) => {
                  return (
                    <TreeNode title={this.renderVIP(value)} key={`${key}`}>
                      {Object.entries(value.servers).map(([skey, svalue]) => {
                        return (
                          <TreeNode
                            title={this.renderServer(value.active, svalue)}
                            key={`${key}-${skey}`}
                            icon={<Icon type="laptop" />}
                          />
                        );
                      })}
                    </TreeNode>
                  );
                })}
              </Tree>
            </Col>
            <Col xs={{ span: 24 }} sm={{ span: 16 }}>
              {this.renderDetail(tree_select)}
            </Col>
          </Row>
        ) : null}
      </div>
    );
  }
}

export default NewDiagram;
