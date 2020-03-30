import React from "react";
import { Tag } from "antd";

const colorTag = {
  CREATE: "blue",
  DELETE: "red",
  UPDATE: "red",
  OWNERSHIP: "blue"
};

const RequestShortDesc = ({ record }) => (
  <div>
    {record.items.map((item, num) => {
      return (
        <div key={num}>
          <Tag color={colorTag[item.CRUD]} style={{ cursor: "text" }}>
            {item.CRUD}
          </Tag>
          &nbsp;
          {item.app.vip}:{item.app.port}&nbsp;
        </div>
      );
    })}
  </div>
);

export default RequestShortDesc;
