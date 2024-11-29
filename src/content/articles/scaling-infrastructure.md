---
title: Scaling Infrastructure
description: Having to scale infrastructure to accommodate growing usage in a distributed application is a common challenge both in practice and in engineering interviews designed to test your understanding.
pubDate: 2024-11-29
---

Many of us however do not necessarily have war stories to think back on, and must rely on theory. So how can we develop heuristics for tackling these challenges when they do arise? Thankfully, there is a wealth of resources to consult.

<div style="max-width: 1200px; margin: 0 auto; font-family: system-ui, -apple-system, sans-serif;">
  <table style="width: 100%; border-collapse: collapse; margin: 20px 0;">
    <thead>
      <tr style="background-color: #f3f4f6;">
        <th style="padding: 12px; text-align: left; border-bottom: 2px solid #e5e7eb;">Active Users (Monthly/Daily)</th>
        <th style="padding: 12px; text-align: left; border-bottom: 2px solid #e5e7eb;">Peak Req/sec</th>
        <th style="padding: 12px; text-align: left; border-bottom: 2px solid #e5e7eb;">Typical Setup</th>
        <th style="padding: 12px; text-align: left; border-bottom: 2px solid #e5e7eb;">Estimated Cost</th>
        <th style="padding: 12px; text-align: left; border-bottom: 2px solid #e5e7eb;">Notes</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">50k / < 10k</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">10-50</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">Single t3.small</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">$20/month</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">Good for MVPs, single instance can handle load</td>
      </tr>
      <tr style="background-color: #f9fafb;">
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">250k / 10k-50k</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">50-250</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">t3.xlarge</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">$150/month</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">Vertical scaling still effective</td>
      </tr>
      <tr>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">1M / 50k-200k</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">250-1000</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">2-3 t3.xlarge behind LB</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">$400/month</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">Start horizontal scaling</td>
      </tr>
      <tr style="background-color: #f9fafb;">
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">5M / 200k-1M</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">1000-5000</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">5-8 t3.xlarge behind LB</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">$1000/month</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">Need robust monitoring, caching</td>
      </tr>
      <tr>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">25M+ / 1M+</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">5000+</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">Custom architecture</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">$2000+/month</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">Need microservices, CDN, etc.</td>
      </tr>
    </tbody>
  </table>

  <div style="background-color: #f3f4f6; padding: 16px; border-radius: 4px; margin-top: 20px;">
    <p style="margin: 0 0 12px 0;"><strong>Assumptions:</strong></p>
    <ul style="margin: 0; padding-left: 20px;">
      <li>1% of daily users are concurrent at peak</li>
      <li>Each active user makes ~1 request/minute at peak</li>
      <li>~20% of monthly users are daily users</li>
      <li>Normal web app complexity (some DB queries, business logic)</li>
      <li>Cost estimates include only compute (EC2), not databases, storage, etc.</li>
      <li>At larger scales, architecture becomes very specific to use case</li>
    </ul>
  </div>
</div>

<div style="max-width: 1200px; margin: 0 auto; font-family: system-ui, -apple-system, sans-serif;">
  <table style="width: 100%; border-collapse: collapse; margin: 20px 0;">
    <thead>
      <tr style="background-color: #f3f4f6;">
        <th style="padding: 12px; text-align: left; border-bottom: 2px solid #e5e7eb;">Application Example</th>
        <th style="padding: 12px; text-align: left; border-bottom: 2px solid #e5e7eb;">MAU/DAU</th>
        <th style="padding: 12px; text-align: left; border-bottom: 2px solid #e5e7eb;">Peak Req/sec</th>
        <th style="padding: 12px; text-align: left; border-bottom: 2px solid #e5e7eb;">Infrastructure Needs</th>
        <th style="padding: 12px; text-align: left; border-bottom: 2px solid #e5e7eb;">System Characteristics</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;"><strong>Personal Blog</strong></td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">5k/1k</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">5-10</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">1 t3.small</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">
          <ul style="margin: 0; padding-left: 20px;">
            <li>Mostly static content</li>
            <li>Occasional DB writes for comments</li>
            <li>Can be handled by single instance</li>
          </ul>
        </td>
      </tr>
      <tr style="background-color: #f9fafb;">
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;"><strong>Small E-commerce</strong></td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">100k/20k</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">100-200</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">2 t3.xlarge</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">
          <ul style="margin: 0; padding-left: 20px;">
            <li>Mix of static/dynamic content</li>
            <li>Product catalog (~10k items)</li>
            <li>Order processing (~100/day)</li>
            <li>Session management critical</li>
          </ul>
        </td>
      </tr>
      <tr>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;"><strong>Medium Forum</strong></td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">500k/100k</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">500-1000</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">4-5 t3.2xlarge</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">
          <ul style="margin: 0; padding-left: 20px;">
            <li>Heavy read/write ratio</li>
            <li>User-generated content</li>
            <li>Search functionality</li>
            <li>Need connection pooling</li>
          </ul>
        </td>
      </tr>
      <tr style="background-color: #f9fafb;">
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;"><strong>Social Media App</strong></td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">2M/400k</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">2000-3000</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">8-10 t3.2xlarge</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">
          <ul style="margin: 0; padding-left: 20px;">
            <li>Real-time updates</li>
            <li>Heavy media storage</li>
            <li>Complex feed generation</li>
            <li>Need CDN + caching</li>
          </ul>
        </td>
      </tr>
      <tr>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;"><strong>Video Streaming</strong></td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">5M/1M</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">5000+</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">Custom Architecture</td>
        <td style="padding: 12px; border-bottom: 1px solid #e5e7eb;">
          <ul style="margin: 0; padding-left: 20px;">
            <li>Massive bandwidth needs</li>
            <li>CDN essential</li>
            <li>Transcoding workers</li>
            <li>Complex caching strategies</li>
          </ul>
        </td>
      </tr>
    </tbody>
  </table>
  
  <div style="background-color: #f3f4f6; padding: 16px; border-radius: 4px; margin-top: 20px;">
    <p style="margin: 0 0 12px 0;"><strong>Notes:</strong></p>
    <ul style="margin: 0; padding-left: 20px;">
      <li>Assumes modern cloud architecture (e.g., AWS)</li>
      <li>Proper caching implemented (Redis/Memcached)</li>
      <li>Basic optimizations in place</li>
      <li>Does not include DB/storage infrastructure</li>
      <li>Social Media assumes ~5 req/min per active user</li>
      <li>Video streaming traffic primarily handled by CDN</li>
    </ul>
  </div>
</div>
